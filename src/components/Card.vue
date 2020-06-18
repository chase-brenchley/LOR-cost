<template>
  <div class="card">
    <h1>{{ card.name }}</h1>
    <img :src="card.assets[0].gameAbsolutePath" alt="An image of a card" />

    <v-row align="center" justify="center">
      <v-btn-toggle v-model="card.quantity" v-on:change="$emit('updateQuantity')" mandatory>
        <v-btn>0</v-btn>
        <v-btn>1</v-btn>
        <v-btn>2</v-btn>
        <v-btn>3</v-btn>
      </v-btn-toggle>
    </v-row>
  </div>
</template>

<script>
export default {
  name: "Card",
  props: {
    card: Object,
    amount: Number
  },
  data() {
    return {
      toggle_exclusive: undefined
    };
  }
};
</script>

<style lang='scss' scoped>
.card {
  width: 25%;
  display: inline-block;
  text-align: center;
  margin: 10px 0;

  img {
    max-width: 100%;
  }
}
</style>