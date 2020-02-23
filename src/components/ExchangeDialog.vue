<template>
  <div>
    <v-card-title v-if="destinationCoin.amount" class="mx-3">
      <span class="font-weight-light title">
        {{ this.depositCoin.amount }}
        {{ this.depositCoin.selected.symbol.toUpperCase() }}
        <v-icon class="mx-2">mdi-arrow-right</v-icon>
        ≈
        {{ this.destinationCoin.amount }}
        {{ this.destinationCoin.selected.symbol.toUpperCase() }}
      </span>
      <v-row justify="end">
        <v-btn absolute top icon @click="cancelOrder">
          <v-icon top>mdi-close</v-icon>
        </v-btn>
      </v-row>
    </v-card-title>
    <div id="step-one" v-if="orderDetails.confirmed === false" class="mx-3">
      <!-- TODO: add arrow icon pointing right -->

      <v-card-text class="font-weight-light">
        <!-- TODO: fix form validation for address -->
        <v-text-field
          v-model="orderDetails.destinationAddress"
          :label="'Enter ' + this.destinationCoin.selected.name + ' Address'"
          required
        ></v-text-field>
        <div>
          <v-checkbox v-model="dialog.terms" color="green">
            <template v-slot:label>
              <div>
                Do you accept the
                <a href="javascript:;" @click.stop="openPage(termsPath)"
                  >terms</a
                >
                and
                <a href="javascript:;" @click.stop="openPage(conditionsPath)"
                  >conditions</a
                >?
              </div>
            </template>
          </v-checkbox>
          <v-btn
            :disabled="!dialog.terms"
            color="primary"
            class="mt-4 px-5"
            height="45px"
            width="100%"
            :loading="orderDetails.loading"
            @click="$emit('exchange')"
          >
            Exchange
          </v-btn>
        </div>
      </v-card-text>
    </div>
    <div id="step-two" v-if="orderDetails.confirmed" class="mx-3">
      <!-- TODO: add arrow icon pointing right -->

      <v-card-text class="font-weight-light text-center">
        <div class="headline mb-3">
          <span class="font-weight-light">Send</span>
          <span class="font-weight-thick" style="color:#76FF03">{{
            ' ' +
              depositCoin.amount +
              ' ' +
              depositCoin.selected.symbol.toUpperCase()
          }}</span>
        </div>
        <p class="mb-0">
          Copy and paste the address, or scan the QR code from your
          {{ depositCoin.selected.symbol.toUpperCase() }} wallet
        </p>
        <div>
          <v-chip class="my-2">
            <v-progress-circular
              indeterminate
              color="primary"
              width="2"
              size="17"
              class="mr-2"
            ></v-progress-circular
            >{{ this.orderDetails.status }}
          </v-chip>
        </div>

        <qrcode
          :value="orderDetails.exchangeAddress"
          :options="{ margin: 1, scale: 5 }"
        ></qrcode>
        <div class="mx-auto mt-5" style="max-width:500px">
          <CopyField
            :id="'1'"
            :orderDetails="orderDetails"
            :vmodel="orderDetails.exchangeAddress"
            :filled="true"
            :dense="true"
            class="mb-4"
          />

          <CopyField
            :id="'2'"
            :orderDetails="orderDetails"
            :vmodel="orderDetails.destinationAddress"
            :label="'RECIPIENT ADDRESS'"
          />

          <CopyField
            :id="'3'"
            :orderDetails="orderDetails"
            :vmodel="orderDetails.orderId"
            :label="'ORDER ID'"
          />
        </div>
      </v-card-text>
    </div>
  </div>
</template>

<script>
import CopyField from './CopyField'

export default {
  components: {
    CopyField
  },
  props: {
    dialog: Object,
    depositCoin: Object,
    destinationCoin: Object,
    orderDetails: Object
  },
  data() {
    return {
      termsPath: 'https//google.com',
      conditionsPath: 'https//google.com/hi'
    }
  },
  methods: {
    openPage(url) {
      window.open(url, '_blank')
    },
    cancelOrder() {
      this.dialog.state = false
      this.orderDetails.confirmed = false
      this.orderDetails.destinationAddress = ''
      this.orderDetails.exchangeAddress = ''
      this.dialog.terms = false
    }
  }
}
</script>
