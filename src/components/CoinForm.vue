<template>
  <v-row no-gutters>
    <v-col cols="6">
      <v-text-field
        type="number"
        height="55px"
        v-model="defaultAmount"
        :label="this.inputType + ' ' + this.selectedCoin.name"
        placeholder="Enter Amount"
        class="pt-0 mt-0"
        :loading="loading"
        hide-details
        outlined
      ></v-text-field>
    </v-col>
    <v-col cols="6" class="pl-2">
      <v-dialog
        v-model="dialog"
        fullscreen
        hide-overlay
        transition="dialog-bottom-transition"
      >
        <template v-slot:activator="{ on }">
          <v-btn color="grey" height="56px" v-on="on" block outlined
            ><v-spacer></v-spacer
            ><v-img
              class="mr-3"
              style="max-width:23px; border-radius:50%"
              :src="selectedCoin.logoUrl"
            ></v-img
            >{{ selectedCoin.symbol }}<v-spacer /><v-icon right
              >mdi-chevron-down</v-icon
            ></v-btn
          >
        </template>
        <v-card width="90%" class="mx-auto">
          <v-toolbar dark color="primary">
            <v-btn icon dark @click="dialog = false">
              <v-icon>mdi-close</v-icon>
            </v-btn>
          </v-toolbar>
          <v-text-field
            v-model="searchCoin"
            class="mx-8"
            placeholder="Seach..."
          ></v-text-field>
          <v-card class="mx-auto" max-width="300" tile>
            <v-list flat height="70vh" style="overflow-y:scroll">
              <v-subheader>Available Coins</v-subheader>
              <v-list-item-group color="primary">
                <v-list-item
                  @click="selectCoin(coin)"
                  v-for="(coin, i) in filterCoins"
                  :key="i"
                >
                  <v-img
                    class="mr-3"
                    style="max-width:30px; border-radius:50%"
                    :src="coin.logoUrl"
                  ></v-img>
                  <v-list-item-content>
                    <v-list-item-title
                      class="ml-4"
                      style="text-transform:uppercase"
                      v-text="coin.symbol"
                    ></v-list-item-title>
                  </v-list-item-content>
                  <v-list-item-content>
                    <v-list-item-title v-text="coin.name"></v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </v-list-item-group>
            </v-list>
          </v-card>
          <v-card class="mx-5 mt-6"> </v-card>
        </v-card>
      </v-dialog>
    </v-col>
    <v-row v-if="this.inputType == 'Send'">
      <v-col class="pb-0 pl-6 pt-4"> </v-col>
    </v-row>
  </v-row>
</template>

<script>
export default {
  name: 'CoinForm',
  props: {
    inputType: String,
    loading: {
      type: Boolean,
      default: function() {
        return false
      }
    },
    coins: {
      type: Array,
      default: function() {
        return []
      }
    }
  },

  data: () => ({
    dialog: false,
    searchCoin: '',
    defaultAmount: {
      type: Number,
      default: function() {
        return null
      }
    },
    selectedCoin: {
      type: Object,
      default: function() {
        return {}
      }
    }
  }),
  computed: {
    filterCoins() {
      let searchCoin = this.searchCoin
      return this.coins.filter(function(coin) {
        return (
          coin.isActive === true &&
          coin.isFiat === false &&
          coin.name.toLowerCase().indexOf(searchCoin.toLowerCase()) > -1
        )
      })
    }
  },
  methods: {
    selectCoin: function(coin) {
      this.selectedCoin = coin
      this.dialog = false
    }
  },
  created() {
    if (this.inputType === 'Send') {
      this.selectedCoin = {
        name: 'Bitcoin',
        symbol: 'BTC',
        logoUrl: 'https://files.coinswitch.co/public/coins/btc.png'
      }
      this.defaultAmount = 0.1
    } else {
      this.selectedCoin = {
        name: 'Ethereum',
        symbol: 'ETH',
        logoUrl: 'https://files.coinswitch.co/public/coins/eth.png'
      }
    }
  }
}
</script>
<style scoped></style>
