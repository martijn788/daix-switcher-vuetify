<template>
  <v-container>
    <v-row justify="center">
      <v-col class="mt-5" sm="8" cols="12">
        <!-- TODO: delete "light" once figured out the right coloring in vuetify.js -->
        <v-card light class="sm-px-4 px-4 sm-py-6 py-10">
          <CoinForm
            :coins="coins"
            :exchangeProps="depositCoin"
            @select-coin="selectCoinDepo"
            v-model.number="depositCoin.amount"
            class=""
          />
          <div class="text-right">
            <v-btn text icon @click="swapCoins">
              <!-- TODO: make swap-vertical icon spin once on click -->
              <v-icon size="30">mdi-swap-vertical</v-icon>
            </v-btn>
          </div>

          <CoinForm
            :coins="coins"
            :exchangeProps="destinationCoin"
            @select-coin="selectCoinDesti"
            v-model.number="destinationCoin.amount"
            class="mt-3"
          />
        </v-card>
        <div style="display:flex; justify-content:center">
          <v-alert
            v-if="error.badAmountErr.state && !error.pairOffline.state"
            class="mt-4 text-center"
            color="red accent-2"
            >{{ error.badAmountErr.msg }}</v-alert
          ><v-alert
            v-if="error.pairOffline.state"
            class="mt-4 text-center"
            color="red accent-2"
            >This trading pair is currently not available.</v-alert
          >
        </div>

        <v-dialog
          v-model="dialog"
          fullscreen
          hide-overlay
          transition="dialog-bottom-transition"
        >
          <template v-slot:activator="{ on }">
            <div class="text-center">
              <v-btn
                :disabled="error.badAmountErr.state || error.pairOffline.state"
                color="primary"
                class="mt-4 px-5"
                height="45px"
                v-on="on"
              >
                Proceed
              </v-btn>
            </div>
          </template>
          <v-card class="mx-auto mt-5">
            <div class="text-right">
              <v-btn icon @click="dialog = false" class="ma-2">
                <v-icon>mdi-close</v-icon>
              </v-btn>
            </div>
          </v-card>
        </v-dialog>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'
import CoinForm from '../components/CoinForm'

import _ from 'lodash'

const API_URL = 'https://api.coinswitch.co/v2/'
const API_KEY = 'Y72jsy9iwD5uiazajayqp190dhjabn3kjauis'

export default {
  components: {
    CoinForm
  },
  name: 'Exchange',
  data() {
    return {
      dialog: false,
      coins: [],
      limit: Object,
      depositCoin: {
        name: 'depositCoin',
        selected: Object,
        amount: 0.1,
        label: 'Send'
      },
      destinationCoin: {
        name: 'destinationCoin',
        selected: Object,
        amount: null,
        label: 'Get',
        disabled: true,
        loading: false
      },
      error: {
        badAmountErr: {
          state: false,
          msg: String
        },
        pairOffline: {
          state: false,
          msg: String
        }
      }
    }
  },
  methods: {
    selectCoinDepo(coin) {
      this.depositCoin.selected = coin
    },
    selectCoinDesti(coin) {
      this.destinationCoin.selected = coin
    },
    setCoin() {
      // Get default coin object from coins
      this.depositCoin.selected = this.coins.find(coin => {
        return coin.symbol === 'btc'
      })
      this.destinationCoin.selected = this.coins.find(coin => {
        return coin.symbol === 'eth'
      })
    },
    calcRate(response) {
      return (
        this.depositCoin.amount * response.data.data.rate -
        response.data.data.minerFee
      )
    },
    swapCoins() {
      this.depositCoin.selected = [
        this.destinationCoin.selected,
        (this.destinationCoin.selected = this.depositCoin.selected)
      ][0]
    },
    checkLimit(value, min, max, coin) {
      if (value < min) {
        this.error.badAmountErr.msg = `Minimum amount is ${min} ${coin.toUpperCase()}`
        this.error.badAmountErr.state = true
      } else if (value > max) {
        this.error.badAmountErr.msg = `Maximum amount is ${max} ${coin.toUpperCase()}`
        this.error.badAmountErr.state = true
      } else {
        this.error.badAmountErr.state = false
      }
    },

    // AXIOS Get ALL Coins
    getCoin() {
      axios({
        method: 'get',
        url: API_URL + 'coins',
        headers: {
          'x-api-key': API_KEY
        }
      })
        .then(response => {
          this.coins = response.data.data
        })
        .then(this.setCoin)
    },
    // AXIOS post rate and validate input
    getRate() {
      let depo = this.depositCoin
      let dest = this.destinationCoin

      dest.loading = true
      dest.amount = 0
      axios({
        method: 'post',
        url: API_URL + 'rate',
        headers: {
          'x-api-key': API_KEY
        },
        data: {
          depositCoin: depo.selected.symbol,
          destinationCoin: dest.selected.symbol
        }
      }).then(response => {
        this.checkAvailable()
        dest.amount = _.round(this.calcRate(response), 6)
        dest.loading = false
        this.limit = response.data.data

        this.checkLimit(
          this.depositCoin.amount,
          this.limit.limitMinDepositCoin,
          this.limit.limitMaxDepositCoin,
          depo.selected.symbol
        )
      })
    },
    checkAvailable() {
      let depo = this.depositCoin
      let dest = this.destinationCoin
      axios({
        method: 'post',
        url: API_URL + 'pairs',
        headers: {
          'x-api-key': API_KEY
        },
        data: {
          depositCoin: depo.selected.symbol,
          destinationCoin: dest.selected.symbol
        }
      }).then(response => {
        console.log(response)
        if (response.data.data.length === 0) {
          this.error.pairOffline.state = true
        } else {
          this.error.pairOffline.state = false
        }
      })
    }
  },
  mounted() {
    this.getCoin()
  },
  watch: {
    depositCoin: {
      handler: function() {
        if (this.depositCoin.amount != '' || this.depositCoin.amount !== 0) {
          this.debouncedGetRate()
        }
      },
      deep: true
    },
    'destinationCoin.selected': {
      handler: function() {
        this.debouncedGetRate()
      }
    }
  },
  created() {
    this.debouncedGetRate = _.debounce(this.getRate, 2500)
  }
}
</script>
