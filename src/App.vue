<template>
  <v-app>
    <v-main>
      <v-text-field
        class="mx-auto my-1 pa-2"
        label="Deck Code"
        v-model="deckCode"
        autofocus
        clearable
        outlined
      ></v-text-field>

      <v-simple-table v-if="neededRarities()" class="ma-2">
        <template v-slot:default>
          <thead>
            <tr>
              <th class="text-left">Rarity</th>
              <th class="text-left">Amount Needed</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="rarity in neededRarities()" :key="rarity.name">
              <td>{{ rarity.name }}</td>
              <td>{{ rarity.amount }}</td>
            </tr>
          </tbody>
        </template>
      </v-simple-table>

      <v-data-table
        :headers="headers"
        :items="neededCards()"
        item-key="cardCode"
        sort-by="name"
        class="ma-2"
        v-if="deckCode && neededCards()"
        group-by="rarity"
        show-group-by
        hide-default-footer
        disable-pagination
      ></v-data-table>

      <div class="card-container">
        <Card
          v-for="card in allCards"
          :key="card.cardCode"
          :card="card"
          v-on:updateQuantity="onUpdateQuantity"
        />
      </div>

      <v-dialog v-model="dialog" max-width="400">
        <v-card>
          <v-card-title class="headline">First time?</v-card-title>
          <v-card-text>Before calculating the cost of a deck, set up your collection!</v-card-text>
        </v-card>
      </v-dialog>
    </v-main>
  </v-app>
</template>

<script>
import { DeckEncoder } from "runeterra";
import Card from "./components/Card";
import set1 from "./data/set1-en_us.json";
import set2 from "./data/set2-en_us.json";

const collection = JSON.parse(localStorage.getItem("collection"));

const byMana = (a, b) => {
  if (a.cost < b.cost) {
    return -1;
  } else if (a.cost > b.cost) {
    return 1;
  } else if (a.name < b.name) {
    return -1;
  } else if (a.name > b.name) {
    return 1;
  }
  return 0;
};

export default {
  name: "App",

  components: {
    Card
  },

  data: () => ({
    deckCode: "",
    // collection: collection,
    headers: [
      { text: "Name", align: "start", value: "name", groupable: false },
      { text: "Amount", value: "quantity", align: "right" },
      { text: "Rarity", value: "rarity", align: "right" }
      // { text: 'Cost', value: 'cost', align: 'right' }
    ],
    dialog: !JSON.parse(localStorage.getItem("collection"))
  }),

  computed: {
    allCards: () => {
      // Create a sorted list of champions
      const cardList = [...set1, ...set2]
        .map(card => {
          const savedCard = collection && collection.find(
            collectionCard => collectionCard.cardCode === card.cardCode
          );

          card.quantity = savedCard ? savedCard.quantity : 0;
          return card;
        })
        .filter(card => card.collectible);
      // .filter(card => !card.cardCode.includes("T"))
      // .filter(card => card.type !== "Trap");

      const champList = cardList
        .filter(card => card.supertype === "Champion")
        .sort(byMana);

      const nonChampList = cardList
        .filter(card => card.supertype !== "Champion")
        .sort(byMana);

      return [...champList, ...nonChampList];
    },
    collection: () => { return JSON.parse(localStorage.getItem("collection")) },
  },

  methods: {
    onUpdateQuantity() {
      // Store new collection to localstorage
      localStorage.setItem("collection", JSON.stringify(this.allCards));
    },
    decodeDeckCode() {
      try {
        return DeckEncoder.decode(this.deckCode);
      } catch (error) {
        return null;
      }
    },
    neededCards() {
      const deck = this.decodeDeckCode();

      if (!deck) {
        return null;
      }

      let neededCards = [];

      deck.forEach(card => {
        const foundCard = this.allCards.find(
          eachCard => card.code === eachCard.cardCode
        );
        const neededCard = Object.assign({}, foundCard);
        neededCard.quantity = neededCard.quantity - card.count;
        neededCard.rarity = neededCard.rarity.toLowerCase();

        if (neededCard.quantity < 0) {
          neededCard.quantity = Math.abs(neededCard.quantity);
          neededCards.push(neededCard);
        }
      });

      return neededCards;
    },
    neededRarities() {
      const neededCards = this.neededCards();
      if (!neededCards) {
        return null;
      }

      return [
        {
          name: "common",
          amount: neededCards
            .filter(card => card.rarity === "common")
            .map(card => card.quantity)
            .reduce((a, b) => a + b, 0)
        },
        {
          name: "rare",
          amount: neededCards
            .filter(card => card.rarity === "rare")
            .map(card => card.quantity)
            .reduce((a, b) => a + b, 0)
        },
        {
          name: "epic",
          amount: neededCards
            .filter(card => card.rarity === "epic")
            .map(card => card.quantity)
            .reduce((a, b) => a + b, 0)
        },
        {
          name: "champion",
          amount: neededCards
            .filter(card => card.rarity === "champion")
            .map(card => card.quantity)
            .reduce((a, b) => a + b, 0)
        },
        {
          name: "total",
          amount: neededCards
            .map(card => card.quantity)
            .reduce((a, b) => a + b, 0)
        }
      ];
    }
  }
};
</script>

<style lang="scss" scoped>
.v-main {
  max-width: 1140px;
  margin: auto;
}
</style>
