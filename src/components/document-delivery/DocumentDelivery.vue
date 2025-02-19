<template>
  <div id="document-delivery">
    <!-- Contact (required) -->
    <v-row no-gutters>
      <v-col cols="12" sm="3" class="pr-4">
        <label class="title-label">{{contactLabel}}</label>
      </v-col>
      <v-col cols="12" sm="9">
        <span id="contact-value">{{contactValue || '(Not entered)'}}</span>
      </v-col>
    </v-row>

    <!-- Custodian of Records (optional) -->
    <v-row no-gutters v-if="showCustodianEmail" class="pt-5">
      <v-col cols="12" sm="3" class="pr-4">
        <label class="title-label">Custodian of Records</label>
      </v-col>
      <v-col cols="12" sm="9">
        <span id="custodian-email">{{custodianEmail || '(Not entered)'}}</span>
      </v-col>
    </v-row>

    <!-- Additional (optional) -->
    <v-row no-gutters v-if="additionalLabel" class="pt-5">
      <v-col cols="12" sm="3" class="pr-4">
        <label class="title-label">{{additionalLabel}}</label>
      </v-col>
      <v-col cols="12" sm="9">
        <span id="additional-value">{{additionalValue || '(Not entered)'}}</span>
      </v-col>
    </v-row>

    <!-- Completing Party (optional) -->
    <v-row no-gutters v-if="showCompletingParty" class="pt-5">
      <v-col cols="12" sm="3" class="pr-4">
        <label class="title-label" :class="{ 'error-text': invalidSection }">
          Completing Party
        </label>
      </v-col>
      <v-col v-if="editableCompletingParty" cols="12" sm="9">
        <v-text-field
          filled persistent-hint validate-on-blur
          id="optionalEmail"
          class="text-input-field"
          label="Client Email Address (Optional)"
          hint="Example: name@email.com"
          v-model="optionalEmail"
          :rules="entityEmailRules"
        />
      </v-col>
      <v-col v-else cols="12" sm="9">
        <span id="completing-party-email">{{completingPartyEmail || '(Not entered)'}}</span>
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { Component, Emit, Prop, Watch } from 'vue-property-decorator'

@Component({})
export default class DocumentDelivery extends Vue {
  // Contact props
  @Prop({ required: true }) readonly contactLabel!: string
  @Prop({ default: null }) readonly contactValue!: string

  // Custodian of Records props
  @Prop({ default: false }) readonly showCustodianEmail!: boolean
  @Prop({ default: null }) readonly custodianEmail!: string

  // Additional props
  @Prop({ default: null }) readonly additionalLabel!: string
  @Prop({ default: null }) readonly additionalValue!: string

  // Completing Party props
  @Prop({ default: true }) readonly showCompletingParty!: boolean
  @Prop({ default: false }) readonly editableCompletingParty!: boolean
  @Prop({ default: null }) readonly completingPartyEmail!: string
  @Prop({ default: null }) readonly documentOptionalEmail!: string

  /** Whether to display invalid section styling. */
  @Prop({ default: false }) readonly invalidSection!: boolean

  // Local properties
  protected optionalEmail = ''

  readonly entityEmailRules = [
    (v: string) => !/^\s/g.test(v) || 'Invalid spaces', // leading spaces
    (v: string) => !/\s$/g.test(v) || 'Invalid spaces', // trailing spaces
    (v: string) => this.validateEmailFormat(v) || 'Enter valid email address'
  ]

  /** Called when component is mounted. */
  mounted (): void {
    this.optionalEmail = this.documentOptionalEmail
  }

  private validateEmailFormat (value: string): boolean {
    // allow empty as the email is optional
    if (!value) {
      return true
    } else {
      // eslint-disable-next-line max-len
      const VALID_FORMAT = new RegExp(/^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/)
      return VALID_FORMAT.test(value)
    }
  }

  @Watch('optionalEmail')
  private onOptionalEmailChanged (val: string): void {
    if (this.validateEmailFormat(val)) {
      this.emitOptionalEmail()
      this.emitValid(true)
    } else {
      this.emitValid(false)
    }
  }

  @Emit('update:optionalEmail')
  private emitOptionalEmail (): string {
    return this.optionalEmail
  }

  @Emit('valid')
  private emitValid (valid: boolean): boolean {
    return valid
  }
}
</script>

<style lang="scss" scoped>
@import '@/assets/styles/theme.scss';

#document-delivery {
  font-size: $px-16;
  color: $gray7;
}

.title-label {
  color: $gray9;
  font-weight: bold;
}

// remove extra space taken by error message
:deep(.v-text-field__details) {
  margin-bottom: -8px !important;
}
</style>
