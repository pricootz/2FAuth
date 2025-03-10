<template>
    <div>
        <setting-tabs :activeTab="'settings.account'"></setting-tabs>
        <div class="options-tabs">
            <form-wrapper>
                <div v-if="isAdmin" class="notification is-warning">
                    {{ $t('settings.you_are_administrator') }}
                </div>
                <form @submit.prevent="submitProfile" @keydown="formProfile.onKeydown($event)">
                    <div v-if="isRemoteUser" class="notification is-warning has-text-centered" v-html="$t('auth.user_account_controlled_by_proxy')" />
                    <h4 class="title is-4 has-text-grey-light">{{ $t('settings.profile') }}</h4>
                    <fieldset :disabled="isRemoteUser">
                        <form-field :form="formProfile" fieldName="name" :label="$t('auth.forms.name')" :maxLength="255" autofocus />
                        <form-field :form="formProfile" fieldName="email" inputType="email" :label="$t('auth.forms.email')" :maxLength="255" />
                        <form-field :form="formProfile" fieldName="password" inputType="password" :label="$t('auth.forms.current_password.label')" :help="$t('auth.forms.current_password.help')" />
                        <form-buttons :isBusy="formProfile.isBusy" :caption="$t('commons.update')" />
                    </fieldset>
                </form>
                <form @submit.prevent="submitPassword" @keydown="formPassword.onKeydown($event)">
                    <h4 class="title is-4 pt-6 has-text-grey-light">{{ $t('settings.change_password') }}</h4>
                    <fieldset :disabled="isRemoteUser">
                        <form-password-field :form="formPassword" fieldName="password" :autocomplete="'new-password'" :showRules="true" :label="$t('auth.forms.new_password')" />
                        <form-field :form="formPassword" fieldName="password_confirmation" inputType="password" :autocomplete="'new-password'" :label="$t('auth.forms.confirm_new_password')" />
                        <form-field :form="formPassword" fieldName="currentPassword" inputType="password" :label="$t('auth.forms.current_password.label')" :help="$t('auth.forms.current_password.help')" />
                        <form-buttons :isBusy="formPassword.isBusy" :caption="$t('auth.forms.change_password')" />
                    </fieldset>
                </form>
                <form id="frmDeleteAccount" @submit.prevent="submitDelete" @keydown="formDelete.onKeydown($event)">
                    <h4 class="title is-4 pt-6 has-text-danger">{{ $t('auth.forms.delete_account') }}</h4>
                    <div class="field is-size-7-mobile">
                        {{ $t('auth.forms.delete_your_account_and_reset_all_data')}}
                    </div>
                    <fieldset :disabled="isRemoteUser">
                        <form-field :form="formDelete" fieldName="password" inputType="password" :label="$t('auth.forms.current_password.label')" :help="$t('auth.forms.current_password.help')" />
                        <form-buttons :isBusy="formDelete.isBusy" :caption="$t('auth.forms.delete_your_account')" :submitId="'btnDeleteAccount'" :color="'is-danger'" />
                    </fieldset>
                </form>
            </form-wrapper>
        </div>
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
    </div>
</template>

<script>

    import Form from './../../components/Form'

    export default {
        data(){
            return {
                formProfile: new Form({
                    name : '',
                    email : '',
                    password : '',
                }),
                formPassword: new Form({
                    currentPassword : '',
                    password : '',
                    password_confirmation : '',
                }),
                formDelete: new Form({
                    password : '',
                }),
                isRemoteUser: this.$root.appConfig.proxyAuth,
                isAdmin: false,
            }
        },

        async mounted() {
            const { data } = await this.formProfile.get('/api/v1/user')

            if( data.is_admin === true ) this.isAdmin = true

            this.formProfile.fill(data)
        },

        methods : {
            submitProfile(e) {
                e.preventDefault()

                this.formProfile.put('/user', {returnError: true})
                .then(response => {
                    this.$notify({ type: 'is-success', text: this.$t('auth.forms.profile_saved') })
                })
                .catch(error => {
                    if( error.response.status === 400 ) {

                        this.$notify({ type: 'is-danger', text: error.response.data.message })
                    }
                    else if( error.response.status !== 422 ) {
                        this.$router.push({ name: 'genericError', params: { err: error.response } });
                    }
                });
            },

            submitPassword(e) {
                e.preventDefault()

                this.formPassword.patch('/user/password', {returnError: true})
                .then(response => {

                    this.$notify({ type: 'is-success', text: response.data.message })
                })
                .catch(error => {
                    if( error.response.status === 400 ) {

                        this.$notify({ type: 'is-danger', text: error.response.data.message })
                    }
                    else if( error.response.status !== 422 ) {
                        
                        this.$router.push({ name: 'genericError', params: { err: error.response } });
                    }
                });
            },

            submitDelete(e) {
                e.preventDefault()

                if(confirm(this.$t('auth.confirm.delete_account'))) {
                    
                    this.formDelete.delete('/user', {returnError: true})
                    .then(response => {

                        this.$notify({ type: 'is-success', text: this.$t('auth.forms.user_account_successfully_deleted') })
                        this.$router.push({ name: 'register' });
                    })
                    .catch(error => {
                        if( error.response.status === 400 ) {

                            this.$notify({ type: 'is-danger', text: error.response.data.message })
                        }
                        else if( error.response.status !== 422 ) {
                            
                            this.$router.push({ name: 'genericError', params: { err: error.response } });
                        }
                    });
                }
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