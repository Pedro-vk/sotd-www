<template>
  <div class="item" :class="errors && errors.length > 0 ? '--has-errors' : ''">
      <textarea class="text-area" maxlength="75" :class="teaser.length > 0 ? '--is-filled' : ''" type="text" v-model="teaser" @input="validate"></textarea>
      <label class="label">Teaser description <span class="required">(required)</span></label>
      <span class="remaining-characters">{{ 75 - teaser.length }}</span>
      <ul v-if="warnings && warnings.length > 0" class="warning-list">
        <li v-for="(warning, index) in warnings" :key="index" class="warning-item">{{ warning }}</li>
      </ul>
      <ul v-if="errors && errors.length > 0" class="error-list">
        <li v-for="(error, index) in errors" :key="index" class="error-item">{{ error }}</li>
      </ul>
      <p class="help">A short teaser that is simple and descriptive. Please don't include self-promotion or obvious words such as "blockchain", "decentralized", or "Ethereum"</p>
    </div>
</template>

<script>
  import { dispatchErrors, dispatchWarnings } from '~/helpers/mixins'

  var validationTimer

  export default {
    computed: {
      errors () {
        return this.$store.getters['dapps/form/teaserErrors']
      },
      teaser: {
        get () {
          return this.$store.getters['dapps/form/teaser']
        },
        set (value) {
          const field = {
            name: 'teaser',
            value: value
          }
          this.$store.dispatch('dapps/form/setField', field)
        }
      },
      warnings () {
        return this.$store.getters['dapps/form/teaserWarnings']
      }
    },
    methods: {
      validate () {
        clearTimeout(validationTimer)
        const errors = {
          field: 'teaser',
          data: []
        }
        const warnings = {
          field: 'teaser',
          data: []
        }
        const warningWords = [
          { value: 'blockchain' },
          { value: 'decentralised', isHidden: true },
          { value: 'decentralized' },
          { value: 'ethereum' }
        ]
        validationTimer = setTimeout(() => {
          this.teaser.length > 75 ? errors.data.push(`Teaser can't be longer than 75 characters`) : ''
          this.teaser.length < 4 ? errors.data.push(`Teaser must be longer than 3 characters`) : ''
          var hasWarningWords = warningWords.some((word) => {
            return this.teaser.toLowerCase().indexOf(word.value) !== -1
          })
          if (hasWarningWords === true) {
            let filteredWarningWords = warningWords.filter((word) => {
              return word.isHidden !== true
            })
            let rowLength = filteredWarningWords.length
            let warningWordString = filteredWarningWords.map((word, i) => {
              let string = '"' + word.value + '"'
              if (rowLength === i + 1) {
                string = 'or "' + word.value + '"'
              }
              return string
            }).join(', ')
            warnings.data.push(`Please don't use obvious words such as ` + warningWordString)
          }
          this.dispatchErrors(errors, 'dapps')
          this.dispatchWarnings(warnings, 'dapps')
        }, 750)
      }
    },
    mixins: [dispatchErrors, dispatchWarnings]
  }
</script>
