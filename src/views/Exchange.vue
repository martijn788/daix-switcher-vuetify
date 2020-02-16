<template>
  <v-container>
    <v-row justify="center">
      <v-col class="mt-5" sm="8" cols="12">
        <v-card class="sm-px-4 px-4 sm-py-6 py-4">
          <CoinForm :coins="coins" class="mb-3" inputType="Send" />
          <CoinForm :coins="coins" inputType="Get" />
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'
import CoinForm from '../components/CoinForm'

export default {
  components: {
    CoinForm
  },
  name: 'Exchange',
  data() {
    return {
      coins: []
    }
  },
  created() {
    axios
      .get('https://api.coinswitch.co/v2/coins', {
        headers: {
          'x-api-key': 'Y72jsy9iwD5uiazajayqp190dhjabn3kjauis'
        }
      })
      .then(response => (this.coins = response.data.data))
  }
}
</script>
