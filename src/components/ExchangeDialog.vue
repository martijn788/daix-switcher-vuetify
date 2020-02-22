<template>
  <div>
    <div id="step-one" v-if="orderDetails.success === false" class="mx-3">
      <!-- TODO: add arrow icon pointing right -->
      <v-card-title v-if="destinationCoin.amount" class="">
        <span class="font-weight-light title">
          {{ this.depositCoin.amount }}
          {{ this.depositCoin.selected.symbol.toUpperCase() }}
          <v-icon></v-icon>
          ≈
          {{ this.destinationCoin.amount }}
          {{ this.destinationCoin.selected.symbol.toUpperCase() }}
        </span>
        <v-row justify="end" posi>
          <v-btn top icon @click="exDialog.dialog = false">
            <v-icon top>mdi-close</v-icon>
          </v-btn>
        </v-row>
      </v-card-title>
      <v-card-text class="font-weight-light">
        <!-- TODO: fix form validation for address -->
        <v-text-field
          v-model="exDialog.destinationAddress"
          :label="'Enter ' + this.destinationCoin.selected.name + ' Address'"
          required
        ></v-text-field>
        <div>
          <v-checkbox v-model="exDialog.terms" color="green">
            <template v-slot:label>
              <div>
                Do you accept the
                <a href="javascript:;" @click.stop="openPage(termsPath)"
                  >terms</a
                >
                and
                <a href="javascript:;" @click.stop="openPage(conditionsPath)"
                  >conditions?</a
                >
              </div>
            </template>
          </v-checkbox>
          <v-btn
            :disabled="!exDialog.terms"
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
    <div id="step-two" v-if="orderDetails.success" class="mx-3">
      <!-- TODO: add arrow icon pointing right -->
      <v-card-title v-if="destinationCoin.amount">
        <span class="font-weight-light title">
          {{ this.depositCoin.amount }}
          {{ this.depositCoin.selected.symbol.toUpperCase() }}
          <v-icon></v-icon>
          ≈
          {{ this.destinationCoin.amount }}
          {{ this.destinationCoin.selected.symbol.toUpperCase() }}
        </span>
        <v-row justify="end" posi>
          <v-btn top icon @click="exDialog.dialog = false">
            <v-icon top>mdi-close</v-icon>
          </v-btn>
        </v-row>
      </v-card-title>
      <v-card-text class="font-weight-light text-center">
        <div class="title mb-2">
          <span class="font-weight-light">Send</span>
          <span style="color:#76FF03">{{
            ' ' +
              depositCoin.amount +
              ' ' +
              depositCoin.selected.symbol.toUpperCase()
          }}</span>
        </div>
        <p>
          Copy and paste the address, or scan the QR code from your
          {{ depositCoin.selected.symbol.toUpperCase() }}
        </p>
        <v-tooltip right open-on-click>
          <template v-slot:activator="{ on }">
            <v-text-field
              id="tf1"
              v-model="orderDetails.exchangeAddress"
              append-icon="mdi-content-copy"
              @click:append="copyToClip('tf1')"
              v-on="on"
              readonly
              filled
              dense
            ></v-text-field>
          </template>
          <span>Copied!</span>
        </v-tooltip>
        <v-tooltip right open-on-click>
          <template v-slot:activator="{ on }">
            <v-text-field
              class="mt-4"
              id="tf2"
              v-model="exDialog.destinationAddress"
              label="RECIPIENT ADDRESS"
              append-icon="mdi-content-copy"
              @click:append="copyToClip('tf2')"
              v-on="on"
              readonly
            ></v-text-field>
          </template>
          <span>Copied!</span>
        </v-tooltip>
        <v-tooltip right open-on-click>
          <template v-slot:activator="{ on }">
            <v-text-field
              id="tf3"
              v-model="orderDetails.orderId"
              label="ORDER ID"
              append-icon="mdi-content-copy"
              @click:append="copyToClip('tf3')"
              v-on="on"
              readonly
            ></v-text-field>
          </template>
          <span>Copied!</span>
        </v-tooltip>
      </v-card-text>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    exDialog: Object,
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
    copyToClip(id) {
      let copyText = document.getElementById(id)
      copyText.select()
      copyText.setSelectionRange(0, 99999)
      document.execCommand('copy')
    }
  }
}
</script>
