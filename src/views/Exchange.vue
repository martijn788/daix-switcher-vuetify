<template>
  <v-container>
    <v-row justify="center">
      <v-col class="mt-5" sm="8" cols="12">
        <v-card class="sm-px-4 px-4 sm-py-6 py-4">
          <CoinForm
            :coins="coins"
            :exchangeProps="depositCoin"
            @select-coin="selectCoinDepo"
            v-model.number="depositCoin.amount"
            class=""
          />
          <div class="text-right">
            <v-btn text icon @click="swapCoins">
              <v-icon>mdi-swap-vertical</v-icon>
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
      coins: [],
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
      limit: Object,
      error: {
        badAmountErr: false
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
      console.log(response)
      return (
        this.depositCoin.amount * response.data.data.rate -
        response.data.data.minerFee
      )
    },
    checklimit() {
      let depoAmount = this.depositCoin.amount
      if (
        depoAmount < this.limit.limitMinDepositCoin ||
        depoAmount > this.limit.limitMaxDepositCoin
      ) {
        console.log('Bad Amount')
      }
    },
    swapCoins() {
      this.depositCoin.selected = [
        this.destinationCoin.selected,
        (this.destinationCoin.selected = this.depositCoin.selected)
      ][0]
    },

    //AXIOS Calls
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
        .catch(response => {
          console.log(response.data.msg)
        })
    },
    getRate() {
      this.destinationCoin.loading = true
      axios({
        method: 'post',
        url: API_URL + 'rate',
        headers: {
          'x-api-key': API_KEY
        },
        data: {
          depositCoin: this.depositCoin.selected.symbol,
          destinationCoin: this.destinationCoin.selected.symbol
        }
      })
        .then(response => {
          this.destinationCoin.amount = _.round(this.calcRate(response), 6)
          this.destinationCoin.loading = false
          this.limit = response.data.data
          let depoAmount = this.depositCoin.amount
          let destAmount = this.destinationCoin.amount
          if (
            depoAmount < this.limit.limitMinDepositCoin ||
            depoAmount > this.limit.limitMaxDepositCoin
          ) {
            this.error.badAmountErr = true
          } else if (
            destAmount < this.limit.limitMinDestinationCoin ||
            destAmount > this.limit.limitMaxDestinationCoin
          ) {
            this.error.badAmountErr = true
          } else {
            this.error.badAmountErr = false
          }
        })
        .catch(response => {
          console.log(response.data.msg)
        })
    }
  },
  mounted() {
    this.getCoin()
  },
  watch: {
    depositCoin: {
      handler: function() {
        this.debouncedGetRate()
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
    this.debouncedGetRate = _.debounce(this.getRate, 2000)
  }
}
</script>
