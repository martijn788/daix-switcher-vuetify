<template>
  <v-row no-gutters>
    <v-col cols="6">
      <v-text-field
        type="number"
        height="55px"
        :value="exchangeProps.amount"
        @input="$emit('input', $event)"
        :label="exchangeProps.label + ' ' + this.exchangeProps.selected.name"
        placeholder="Enter Amount"
        class="pt-0 mt-0"
        :readonly="exchangeProps.disabled"
        :loading="exchangeProps.loading"
        :disabled="exchangeProps.loading"
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
              :src="exchangeProps.selected.logoUrl"
            ></v-img
            >{{ exchangeProps.selected.symbol }}<v-spacer /><v-icon right
              >mdi-chevron-down</v-icon
            ></v-btn
          >
        </template>
        <v-card class="mx-auto mt-5">
          <div class="text-right">
            <v-btn icon @click="dialog = false" class="ma-2">
              <v-icon>mdi-close</v-icon>
            </v-btn>
          </div>

          <v-text-field
            autofocus
            v-model="searchCoin"
            class="px-8"
            placeholder="Search.."
          ></v-text-field>
          <v-list flat style="overflow-y:scroll" height="80vh">
            <v-subheader class="pl-8">Available Coins</v-subheader>
            <v-list-item-group color="primary">
              <v-list-item
                @click="selectCoin(coin)"
                v-for="(coin, i) in filterCoins"
                :key="i"
                class="px-8"
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
          <v-card class="mx-5 mt-6"> </v-card>
        </v-card>
      </v-dialog>
    </v-col>
    <v-row v-if="this.exchangeProps.label === 'Send'">
      <v-col class="pb-0 pl-6 pt-4"> </v-col>
    </v-row>
  </v-row>
</template>

<script>
export default {
  name: 'CoinForm',
  props: {
    coins: Array,
    exchangeProps: Object
  },

  data: () => ({
    dialog: false,
    searchCoin: ''
  }),
  computed: {
    filterCoins() {
      let searchCoin = this.searchCoin
      return this.coins.filter(coin => {
        return (
          (coin.isActive === true &&
            coin.isFiat === false &&
            coin.symbol.toLowerCase().indexOf(searchCoin.toLowerCase()) > -1) ||
          (coin.isActive === true &&
            coin.isFiat === false &&
            coin.name.toLowerCase().indexOf(searchCoin.toLowerCase()) > -1)
        )
      })
    }
  },
  methods: {
    selectCoin(coin) {
      this.$emit('select-coin', coin)
      this.dialog = false
    }
  }
}
</script>
<style scoped></style>
