<template>
  <div>
    <v-text-field
      :id="id"
      v-model="vmodel"
      :label="label"
      readonly
      :filled="filled"
      :dense="dense"
      ><v-tooltip
        slot="append"
        top
        color="primary"
        :open-on-click="openclick"
        :open-on-hover="openhover"
      >
        <template v-slot:activator="{ on }">
          <v-icon v-on="on" @click="copyToClip(id)">mdi-content-copy</v-icon>
        </template>
        Copied
      </v-tooltip></v-text-field
    >
  </div>
</template>

<script>
export default {
  props: {
    orderDetails: Object,
    vmodel: String,
    label: String,
    id: String,
    filled: Boolean,
    dense: Boolean
  },
  data() {
    return {
      openhover: false,
      openclick: true
    }
  },
  methods: {
    copyToClip(id) {
      let copyText = document.getElementById(id)
      copyText.select()
      copyText.setSelectionRange(0, 99999)
      document.execCommand('copy')
    }
  }
}
</script>

<style></style>
