<template>
  <v-layout>
    <v-row>
      <v-col cols="12" md="4">
        <v-form>
          <v-text-field
            v-model="formData.amount"
            label="Payment Intent Amount"
          />

          <v-select
            v-model="formData.currency"
            :items="supportedCurrencies"
            label="Payment Currency"
          />

          <v-select
            v-model="formData.blockchain"
            :items="supportedChains"
            label="Blockchain"
          />

          <v-select
            v-model="formData.settlementCurrency"
            :items="supportedCurrencies"
            label="Settlement Currency"
          />

          <v-text-field v-model="formData.expiresOn" label="Expires On" />

          <v-btn
            depressed
            class="mb-7"
            color="primary"
            :loading="loading"
            @click.prevent="makeApiCall"
          >
            Make api call
          </v-btn>
        </v-form>
      </v-col>
      <v-col cols="12" md="8">
        <RequestInfo
          :url="requestUrl"
          :payload="payload"
          :response="response"
        />
      </v-col>
    </v-row>
    <ErrorSheet
      :error="error"
      :show-error="showError"
      @onChange="onErrorSheetClosed"
    />
  </v-layout>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'
import { mapGetters } from 'vuex'
import { v4 as uuidv4 } from 'uuid'
import RequestInfo from '@/components/RequestInfo.vue'
import ErrorSheet from '@/components/ErrorSheet.vue'
import { CreatePaymentIntentPayload } from '@/lib/paymentIntentsApi'
@Component({
  components: {
    RequestInfo,
    ErrorSheet,
  },
  computed: {
    ...mapGetters({
      payload: 'getRequestPayload',
      response: 'getRequestResponse',
      requestUrl: 'getRequestUrl',
      isMarketplace: 'isMarketplace',
    }),
  },
})
export default class CreatePaymentIntentClass extends Vue {
  formData = {
    idempotencyKey: '',
    amount: '0.00',
    currency: '',
    settlementCurrency: '',
    blockchain: '',
    expiresOn: '',
  }

  required = [(v: string) => !!v || 'Field is required']
  error = {}
  loading = false
  showError = false
  supportedCurrencies = ['BTC', 'ETH', 'USD']
  supportedChains = ['BTC', 'ETH']
  onErrorSheetClosed() {
    this.error = {}
    this.showError = false
  }

  async makeApiCall() {
    this.loading = true
    const amountDetail = {
      amount: this.formData.amount,
      currency: this.formData.currency,
    }
    const blockchainPaymentMethod = {
      type: 'blockchain',
      chain: this.formData.blockchain,
    }
    const payload: CreatePaymentIntentPayload = {
      idempotencyKey: uuidv4(),
      amount: amountDetail,
      settlementCurrency: this.formData.settlementCurrency,
      paymentMethods: [blockchainPaymentMethod],
      expiresOn: this.formData.expiresOn,
    }
    try {
      await this.$paymentIntentsApi.createPaymentIntent(payload)
    } catch (error) {
      this.error = error
      this.showError = true
    } finally {
      this.loading = false
    }
  }
}
</script>
