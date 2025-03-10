<template>
    <div>
        <setting-tabs :activeTab="'settings.webauthn.devices'"></setting-tabs>
        <div class="options-tabs">
            <form-wrapper>
                <div v-if="isRemoteUser" class="notification is-warning has-text-centered" v-html="$t('auth.auth_handled_by_proxy')" />
                <h4 class="title is-4 has-text-grey-light">{{ $t('auth.webauthn.security_devices') }}</h4>
                <div class="is-size-7-mobile">
                    {{ $t('auth.webauthn.security_devices_legend')}}
                </div>
                <div class="mt-3">
                    <a tabindex="0" @click="register" @keyup.enter="register">
                        <font-awesome-icon :icon="['fas', 'plus-circle']" />&nbsp;{{ $t('auth.webauthn.register_a_new_device')}}
                    </a>
                </div>
                <!-- credentials list -->
                <div v-if="credentials.length > 0" class="field">
                    <div v-for="credential in credentials" :key="credential.id" class="group-item is-size-5 is-size-6-mobile">
                        {{ displayName(credential) }}
                        <!-- revoke link -->
                        <button class="button tag is-pulled-right" :class="$root.showDarkMode ? 'is-dark':'is-white'" @click="revokeCredential(credential.id)" :title="$t('settings.revoke')">
                            {{ $t('settings.revoke') }}
                        </button>
                        <!-- edit link -->
                        <!-- <router-link :to="{ name: '' }" class="has-text-grey pl-1" :title="$t('commons.rename')">
                            <font-awesome-icon :icon="['fas', 'pen-square']" />
                        </router-link> -->
                    </div>
                    <div class="mt-2 is-size-7 is-pulled-right">
                        {{ $t('auth.webauthn.revoking_a_device_is_permanent')}}
                    </div>
                </div>
                <div v-if="isFetching && credentials.length === 0" class="has-text-centered mt-6">
                    <span class="is-size-4">
                        <font-awesome-icon :icon="['fas', 'spinner']" spin />
                    </span>
                </div>
                <h4 class="title is-4 pt-6 has-text-grey-light">{{ $t('settings.options') }}</h4>
                <div class="field">
                    {{ $t('auth.webauthn.need_a_security_device_to_enable_options')}}
                </div>
                <form>
                    <!-- use webauthn only -->
                    <form-checkbox v-on:useWebauthnOnly="savePreference('useWebauthnOnly', $event)" :form="form" fieldName="useWebauthnOnly" :label="$t('auth.webauthn.use_webauthn_only.label')" :help="$t('auth.webauthn.use_webauthn_only.help')" :disabled="isRemoteUser || credentials.length === 0" />
                </form>
                <!-- footer -->
                <vue-footer :showButtons="true">
                    <!-- close button -->
                    <p class="control">
                        <router-link
                            :to="{ path: $route.params.returnTo, params: { toRefresh: false } }"
                            class="button is-rounded"
                            :class="{'is-dark' : $root.showDarkMode}"
                            tabindex="0"
                            role="button"
                            :aria-label="$t('commons.close_the_x_page', {pagetitle: $router.currentRoute.meta.title})">
                            {{ $t('commons.close') }}
                        </router-link>
                    </p>
                </vue-footer>
            </form-wrapper>
        </div>
    </div>
</template>

<script>

    import Form from './../../components/Form'
    import WebAuthn from './../../components/WebAuthn'

    export default {
        data(){
            return {
                form: new Form({
                    useWebauthnOnly: null,
                }),
                credentials: [],
                isFetching: false,
                isRemoteUser: false,
                webauthn: new WebAuthn()
            }
        },

        async mounted() {

            const { data } = await this.form.get('/api/v1/user/preferences')

            this.form.fillWithKeyValueObject(data)
            this.form.setOriginal()

            this.fetchCredentials()
        },

        methods : {

            /**
             * Save a preference
             */
            savePreference(preferenceName, event) {
                this.axios.put('/api/v1/user/preferences/' + preferenceName, { value: event }).then(response => {
                    this.$notify({ type: 'is-success', text: this.$t('settings.forms.setting_saved') })
                    this.$root.userPreferences[response.data.key] = response.data.value
                })
            },


            /**
             * Get all credentials from backend
             */
            async fetchCredentials() {

                this.isFetching = true

                await this.axios.get('/webauthn/credentials', {returnError: true})
                .then(response => {
                    this.credentials = response.data
                })
                .catch(error => {
                    if( error.response.status === 400 ) {

                        this.isRemoteUser = true
                    }
                    else {

                        this.$router.push({ name: 'genericError', params: { err: error.response } });
                    }
                })

                this.isFetching = false
            },


            /**
             * Register a new security device
             */
            async register() {

                if (this.isRemoteUser) {
                    this.$notify({ type: 'is-warning', text: this.$t('errors.unsupported_with_reverseproxy') })
                    return false
                }

                // Check https context
                if (!window.isSecureContext) {
                    this.$notify({ type: 'is-danger', text: this.$t('errors.https_required') })
                    return false
                }

                // Check browser support
                if (this.webauthn.doesntSupportWebAuthn) {
                    this.$notify({ type: 'is-danger', text: this.$t('errors.browser_does_not_support_webauthn') })
                    return false
                }

                const registerOptions = await this.axios.post('/webauthn/register/options').then(res => res.data)
                const publicKey = this.webauthn.parseIncomingServerOptions(registerOptions)
                let bufferedCredentials

                try {
                    bufferedCredentials = await navigator.credentials.create({ publicKey })
                }
                catch (error) {
                    if (error.name == 'AbortError') {
                        this.$notify({ type: 'is-warning', text: this.$t('errors.aborted_by_user') })
                    }
                    else if (error.name == 'SecurityError') {
                        this.$notify({ type: 'is-danger', text: this.$t('errors.security_error_check_rpid') })
                    }
                    else if (error.name == 'InvalidStateError') {
                        this.$notify({ type: 'is-danger', text: this.$t('errors.security_device_unsupported') })
                    }
                    else if (error.name == 'NotAllowedError') {
                        this.$notify({ type: 'is-danger', text: this.$t('errors.not_allowed_operation') })
                    }
                    else if (error.name == 'NotSupportedError') {
                        this.$notify({ type: 'is-danger', text: this.$t('errors.unsupported_operation') })
                    }
                    return false
                }

                const publicKeyCredential = this.webauthn.parseOutgoingCredentials(bufferedCredentials);

                this.axios.post('/webauthn/register', publicKeyCredential, {returnError: true})
                .then(response => {
                    this.$router.push({ name: 'settings.webauthn.editCredential', params: { id: publicKeyCredential.id, name: this.$t('auth.webauthn.my_device') } })
                })
                .catch(error => {
                    if( error.response.status === 422 ) {
                        this.$notify({ type: 'is-danger', text: error.response.data.message })
                    }
                    else {
                        this.$router.push({ name: 'genericError', params: { err: error.response } });
                    }
                })
            },


            /**
             * revoke a credential
             */
            async revokeCredential(credentialId) {
                if(confirm(this.$t('auth.confirm.revoke_device'))) {

                    await this.axios.delete('/webauthn/credentials/' + credentialId).then(response => {
                        // Remove the revoked credential from the collection
                        this.credentials = this.credentials.filter(a => a.id !== credentialId)

                        if (this.credentials.length == 0) {
                            this.form.useWebauthnOnly = false
                            this.$root.userPreferences['useWebauthnOnly'] = false
                        }

                        this.$notify({ type: 'is-success', text: this.$t('auth.webauthn.device_revoked') })
                    });
                }
            },

            /**
             * Always display a printable name
             */
            displayName(credential) {
                return credential.alias ? credential.alias : this.$t('auth.webauthn.my_device') + ' (#' + credential.id.substring(0, 10) + ')'
            },

        },

        beforeRouteEnter(to, from, next) {
            next(vm => {
                if (from.params.returnTo) {
                    to.params.returnTo = from.params.returnTo
                }
                else {
                    to.params.returnTo = from.name
                        ? from.path
                        : '/accounts'
                }
            })
        },

        beforeRouteLeave(to, from, next) {
            if (to.name == 'accounts') {
                this.$notify({ clean: true })
            }
            next()
        }
    }
</script>