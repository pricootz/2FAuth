<template>
    <div class="error-message">
        <modal v-model="ShowModal" :closable="this.showcloseButton">
            <div class="error-message" v-if="$route.name == '404'">
                <p class="error-404"></p>
                <p>{{ $t('errors.resource_not_found') }}</p>
                <p class=""><router-link :to="{ name: 'accounts' }" class="is-text">{{ $t('errors.refresh') }}</router-link></p>
            </div>
            <div v-else>
                <p class="error-generic"></p>
                <p>{{ $t('errors.error_occured') }} </p>
                <p v-if="error.message" class="has-text-grey-lighter">{{ error.message }}</p>
                <p v-if="error.originalMessage" class="has-text-grey-lighter">{{ error.originalMessage }}</p>
                <p><router-link :to="{ name: 'accounts', params: { toRefresh: true } }" class="is-text">{{ $t('errors.refresh') }}</router-link></p>
                <p v-if="debugMode == 'development' && error.debug">
                    <br>
                    {{ error.debug }}
                </p>
            </div>
        </modal>
    </div>
</template>


<script>
    import Modal from '../components/Modal'

    export default {
        data(){
            return {
                ShowModal : true,
                showcloseButton: this.closable,
            }
        },

        computed: {

            debugMode: function() {
                return process.env.NODE_ENV
            },

            error: function() {
                if( this.err === null || this.err === undefined ) {
                    return false
                }
                else
                {
                    if (this.err.status === 407) {
                        return {
                            'message' : this.$t('errors.auth_proxy_failed'),
                            'originalMessage' : this.$t('errors.auth_proxy_failed_legend')
                        }
                    }
                    else if (this.err.status === 403) {
                        return {
                            'message' : this.$t('errors.unauthorized'),
                            'originalMessage' : this.$t('errors.unauthorized_legend')
                        }
                    }
                    else if(this.err.data) {
                        return this.err.data
                    }
                    else {
                        return { 'message' : this.err }
                    }

                }
            }

        },

        props: {
            err: [String, Object], // on object (error.response) or a string
            closable: {
                type: Boolean,
                default: true
            }
        }, 

        components: {
            Modal
        },

        mounted(){
            // stop OTP generation on modal close
            this.$on('modalClose', function() {
                window.history.length > 1 ? this.$router.go(-1) : this.$router.push({ name: 'accounts '})
            });

        },
    }

</script>

