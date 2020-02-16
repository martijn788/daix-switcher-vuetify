<template>
  <div id="nav">
    <v-app-bar color="primary" app flat dark>
      <v-app-bar-nav-icon @click.stop="drawer = true" dark></v-app-bar-nav-icon>
      <v-spacer></v-spacer>
      <v-toolbar-title dark>DAIX</v-toolbar-title>
    </v-app-bar>
    <v-navigation-drawer v-model="drawer" absolute app temporary dark>
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title>
            DAIX
          </v-list-item-title>
        </v-list-item-content>
        <v-list-item-icon @click.stop="drawer = false" link>
          <v-icon>mdi-close</v-icon>
        </v-list-item-icon>
      </v-list-item>
      <v-list-item :to="view.path" v-for="(view, i) in views" :key="i" link>
        <v-list-item-icon>
          <v-icon>{{ view.icon }}</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-title v-text="view.name"></v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-navigation-drawer>
  </div>
</template>

<script>
export default {
  name: 'NavBar',
  data() {
    return {
      drawer: false,
      views: [
        {
          name: 'Exchange',
          path: '/',
          icon: 'mdi-swap-vertical-bold'
        },
        {
          name: 'Orders',
          path: '/orders',
          icon: 'mdi-history'
        }
      ]
    }
  }
}
</script>

<style></style>
