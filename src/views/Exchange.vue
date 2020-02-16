<template>
  <v-container>
    <v-row justify="center">
      <v-col class="mt-5" sm="8" cols="12">
        <v-card class="sm-px-4 px-4 sm-py-6 py-4">
          <CoinForm
            :coins="coins"
            :exchangeProps="depositCoin"
            @select-coin="selectCoinDepo"
            class="mb-3"
          />
          <CoinForm
            :coins="coins"
            :exchangeProps="destinationCoin"
            @select-coin="selectCoinDesti"
          />
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'
import CoinForm from '../components/CoinForm'

const API_URL = 'https://api.coinswitch.co/v2/'

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
        label: 'Get'
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
    setCoin(response) {
      this.coins = response.data.data
      // Get default coin object from co
      this.depositCoin.selected = this.coins.find(coin => {
        return coin.name === 'Bitcoin'
      })
      this.destinationCoin.selected = this.coins.find(coin => {
        return coin.name === 'Ethereum'
      })
    },
    axiosGetCoins() {
      axios
        .get(API_URL + 'coins', {
          headers: {
            'x-api-key': 'Y72jsy9iwD5uiazajayqp190dhjabn3kjauis'
          }
        })
        .then(response => this.setCoin(response))
    }
  },
  created() {
    this.axiosGetCoins()
  }
}
</script>
