<template>
    <div class="field" :class="{ 'pt-3' : hasOffset }" role="radiogroup" :aria-labelledby="inputId('label', fieldName)">
        <label v-if="label" :id="inputId('label', fieldName)" class="label" v-html="label"></label>
        <div class="is-toggle buttons">
            <button 
                role="radio" 
                type="button"
                class="button" 
                :aria-checked="form[fieldName] === choice.value"
                :disabled="isDisabled" 
                v-for="(choice, index) in choices" 
                :key="index" 
                :class="{
                    'is-link' : form[fieldName] === choice.value,
                    'is-dark' : $root.showDarkMode,
                    'is-multiline' : choice.legend,
                }" 
                v-on:click.stop="setRadio(choice.value)" 
                :title="choice.title ? choice.title : ''"
            >
                <input 
                    :id="inputId(inputType, choice.value)" 
                    :type="inputType" 
                    class="is-hidden" 
                    :checked="form[fieldName] === choice.value" 
                    :value="choice.value" 
                    v-model="form[fieldName]" 
                    :disabled="isDisabled" 
                />
                <span v-if="choice.legend" v-html="choice.legend" class="is-block is-size-7"></span>
                <font-awesome-icon :icon="['fas', choice.icon]" v-if="choice.icon" class="mr-2" /> {{ choice.text }}
            </button>
        </div>
        <field-error :form="form" :field="fieldName" />
        <p class="help" v-html="help" v-if="help"></p>
    </div>
</template>

<script>
    export default {
        name: 'FormToggle',
        
        data() {
            return {
                inputType: 'radio'
            }
        },

        props: {
            label: {
                type: String,
                default: ''
            },

            fieldName: {
                type: String,
                default: '',
                required: true
            },

            choices: {
                type: Array,
                required: true
            },

            form: {
                type: Object,
                required: true
            },

            help: {
                type: String,
                default: ''
            },

            hasOffset: {
                type: Boolean,
                default: false
            },

            isDisabled: {
                type: Boolean,
                default: false
            }
        },

        methods: {
            setRadio(event) {
                this.form[this.fieldName] = event
                this.$emit(this.fieldName, this.form[this.fieldName])
            }
        }
    }
</script>