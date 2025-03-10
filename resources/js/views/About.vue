<template>
    <responsive-width-wrapper>
        <h1 class="title has-text-grey-dark">{{ pagetitle }}</h1>
        <p class="block">
            <span :class="$root.showDarkMode ? 'has-text-white':'has-text-black'"><span class="is-size-5">2FAuth</span> v{{ appVersion }}</span><br />
            {{ $t('commons.2fauth_teaser')}}
        </p>
        <img class="about-logo" src="logo.svg" alt="2FAuth logo" />
        <p class="block">
            ©Bubka <a class="is-size-7" href="https://github.com/Bubka/2FAuth/blob/master/LICENSE">AGPL-3.0 license</a>
        </p>
        <h2 class="title is-5 has-text-grey-light">
            {{ $t('commons.resources') }}
        </h2>
        <div class="buttons">
            <a class="button" :class="{'is-dark' : $root.showDarkMode}" href="https://github.com/Bubka/2FAuth" target="_blank">
                <span class="icon is-small">
                    <font-awesome-icon :icon="['fab', 'github-alt']" />
                </span>
                <span>Github</span>
            </a>
            <a class="button" :class="{'is-dark' : $root.showDarkMode}" href="https://docs.2fauth.app/" target="_blank">
                <span class="icon is-small">
                    <font-awesome-icon :icon="['fas', 'book']" />
                </span>
                <span>Docs</span>
            </a>
            <a class="button" :class="{'is-dark' : $root.showDarkMode}" href="https://demo.2fauth.app/" target="_blank">
                <span class="icon is-small">
                    <font-awesome-icon :icon="['fas', 'flask']" />
                </span>
                <span>Demo</span>
            </a>
            <a class="button" :class="{'is-dark' : $root.showDarkMode}" href="https://docs.2fauth.app/resources/rapidoc.html" target="_blank">
                <span class="icon is-small">
                    <font-awesome-icon :icon="['fas', 'code']" />
                </span>
                <span>API</span>
            </a>
        </div>
        <h2 class="title is-5 has-text-grey-light">
            {{ $t('commons.credits') }}
        </h2>
        <p class="block">
            <ul>
                <li>{{ $t('commons.made_with')}}&nbsp;<a href="https://docs.2fauth.app/credits/">Laravel, Bulma CSS, Vue.js and more</a></li>
                <li>{{ $t('commons.ui_icons_by')}}&nbsp;<a href="https://fontawesome.com/">Font Awesome</a>&nbsp;<a class="is-size-7" href="https://fontawesome.com/license/free">(CC BY 4.0, Fonts: SIL OFL 1.1, Code: MIT License)</a></li>
                <li>{{ $t('commons.logos_by')}}&nbsp;<a href="https://2fa.directory/">2FA Directory</a>&nbsp;<a class="is-size-7" href="https://github.com/2factorauth/twofactorauth/blob/master/LICENSE.md">(MIT License)</a></li>
            </ul>
        </p>
        <h2 class="title is-5 has-text-grey-light">
            {{ $t('commons.environment') }}
        </h2>
        <div class="about-debug box is-family-monospace is-size-7">
            <button :aria-label="$t('commons.copy_to_clipboard')" class="button is-like-text is-pulled-right is-small is-text" v-clipboard="() => this.$refs.listInfos.innerText" v-clipboard:success="clipboardSuccessHandler">
                <font-awesome-icon :icon="['fas', 'copy']" />
            </button>
            <ul ref="listInfos">
                <li v-for="(value, key) in infos" :value="value" :key="key"><b>{{key}}</b>: {{value}}</li>
            </ul>
        </div>
        <h2 v-if="showAdminSettings" class="title is-5 has-text-grey-light">
            {{ $t('settings.admin_settings') }}
        </h2>
        <div v-if="showAdminSettings" class="about-debug box is-family-monospace is-size-7">
            <button :aria-label="$t('commons.copy_to_clipboard')" class="button is-like-text is-pulled-right is-small is-text" v-clipboard="() => this.$refs.listAdminSettings.innerText" v-clipboard:success="clipboardSuccessHandler">
                <font-awesome-icon :icon="['fas', 'copy']" />
            </button>
            <ul ref="listAdminSettings">
                <li v-for="(value, setting) in adminSettings" :value="value" :key="setting"><b>{{setting}}</b>: {{value}}</li>
            </ul>
        </div>
        <h2 v-if="showUserPreferences" class="title is-5 has-text-grey-light">
            {{ $t('settings.user_preferences') }}
        </h2>
        <div v-if="showUserPreferences" class="about-debug box is-family-monospace is-size-7">
            <button :aria-label="$t('commons.copy_to_clipboard')" class="button is-like-text is-pulled-right is-small is-text" v-clipboard="() => this.$refs.listUserPreferences.innerText" v-clipboard:success="clipboardSuccessHandler">
                <font-awesome-icon :icon="['fas', 'copy']" />
            </button>
            <ul ref="listUserPreferences">
                <li v-for="(value, preference) in userPreferences" :value="value" :key="preference"><b>{{preference}}</b>: {{value}}</li>
            </ul>
        </div>
        <!-- footer -->
        <vue-footer :showButtons="true">
            <!-- close button -->
            <p class="control">
                <router-link
                    :to="{ path: $route.params.goBackTo, params: { returnTo: $route.params.returnTo, toRefresh: true } }"
                    :aria-label="$t('commons.close_the_x_page', {pagetitle: pagetitle})"
                    class="button is-rounded"
                    :class="{'is-dark' : $root.showDarkMode}">
                    {{ $t('commons.back') }}
                </router-link>
            </p>
        </vue-footer>
    </responsive-width-wrapper>
</template>

<script>
    export default {
        data() {
            return {
                pagetitle: this.$t('commons.about'),
                infos : null,
                adminSettings : null,
                userPreferences : null,
                showUserPreferences: false,
                showAdminSettings: false,
            }
        },

        async mounted() {
            await this.axios.get('infos').then(response => {
                this.infos = response.data.common

                if (response.data.admin_settings) {
                    this.adminSettings = response.data.admin_settings
                    this.showAdminSettings = true
                }

                if (response.data.user_preferences) {
                    this.userPreferences = response.data.user_preferences
                    this.showUserPreferences = true
                }
            })
        },

        methods: {
            
            clipboardSuccessHandler ({ value, event }) {
                this.$notify({ type: 'is-success', text: this.$t('commons.copied_to_clipboard') })
            },

            clipboardErrorHandler ({ value, event }) {
                console.log('error', value)
            },
        },

        beforeRouteEnter(to, from, next) {
            next(vm => {
                if (from.params.returnTo) {
                    to.params.returnTo = from.params.returnTo
                }

                to.params.goBackTo = from.name
                    ? from.path
                    : '/accounts'
            })
        },

    }
</script>