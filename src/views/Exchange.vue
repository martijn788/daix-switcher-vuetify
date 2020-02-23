<template>
  <v-container>
    <v-row justify="center">
      <v-col class="" sm="8" cols="12">
        <div class="font-weight-light">
          <h1 class="mb-5 pt-10 font-regular" style="line-height:1.05">
            Change Crypto For Better Rates
          </h1>
          <p class="subtitle-1 mb-10" style="line-height:1.2">
            Over 300+ coins without needing to signup for an account.
          </p>
        </div>
        <v-card class="sm-px-4 px-4 sm-py-6 py-10">
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
        <v-dialog
          v-model="dialog.state"
          hide-overlay
          transition="dialog-bottom-transition"
        >
          <template v-slot:activator="{ on }">
            <div class="text-center">
              <v-btn
                :disabled="
                  error.badAmountErr.state ||
                    error.pairOffline.state ||
                    !destinationCoin.amount
                "
                color="primary"
                class="mt-4 px-5"
                height="45px"
                v-on="on"
              >
                Proceed
              </v-btn>
            </div>
          </template>
          <v-card
            class="mx-auto"
            style="border-radius:15px; position: absolute; bottom: 0;"
            width="100vw"
            height="95%"
          >
            <ExchangeDialog
              :dialog="dialog"
              :destinationCoin="destinationCoin"
              :depositCoin="depositCoin"
              :orderDetails="orderDetails"
              @exchange="getOrder"
            />
          </v-card>
        </v-dialog>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'
import CoinForm from '../components/CoinForm'
import ExchangeDialog from '../components/ExchangeDialog'

import _ from 'lodash'

const API_URL = 'https://api.coinswitch.co/v2/'
const API_KEY = 'Y72jsy9iwD5uiazajayqp190dhjabn3kjauis'

export default {
  components: {
    CoinForm,
    ExchangeDialog
  },
  name: 'Exchange',
  data() {
    return {
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
      },
      dialog: {
        terms: false,
        state: false
      },
      orderDetails: {
        exchangeAddress: String,
        orderId: String,
        confirmed: false,
        loading: false,
        destinationAddress: '',
        status: 'Awaiting Deposit'
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

    // API CALLS
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
    getOrder() {
      let depo = this.depositCoin
      let dest = this.destinationCoin

      this.orderDetails.loading = true
      axios({
        method: 'post',
        url: API_URL + 'order',
        headers: {
          'x-api-key': API_KEY
        },
        data: {
          depositCoin: depo.selected.symbol,
          destinationCoin: dest.selected.symbol,
          depositCoinAmount: depo.amount,
          destinationAddress: {
            address: this.orderDetails.destinationAddress,
            tag: null
          }
        }
      }).then(response => {
        let data = response.data.data
        this.orderDetails.orderId = data.orderId
        this.orderDetails.exchangeAddress = data.exchangeAddress.address
        this.orderDetails.confirmed = true
        this.orderDetails.loading = false
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
    'depositCoin.amount': {
      handler: function() {
        if (this.depositCoin.amount != '' || this.depositCoin.amount !== 0) {
          this.debouncedGetRate()
        }
      }
    },
    'destinationCoin.selected': {
      handler: function() {
        this.getRate()
      }
    }
  },
  created() {
    this.debouncedGetRate = _.debounce(this.getRate, 2500)
  }
}
</script>

<style scoped></style>
