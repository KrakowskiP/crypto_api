<template>
  <v-app>
    <v-card>
      <v-card-title>
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Search"
          hide-details
        ></v-text-field>
      </v-card-title>
      <v-data-table
        :headers="headers"
        :items="filteredResponse"
        :disable-sort="true"
        @click:row="getDetailsCoin"
        hide-default-footer
        :items-per-page="itemsPerPage"
      >
        <template v-slot:item.coin="{ item }">
          <div style="display: flex">
            <img :src="item.image" />
            <div class="coin">{{ item.name }}</div>
            <div class="coin gray">{{ item.symbol.toUpperCase() }}</div>
          </div>
        </template>
        <template v-slot:item.rank="{ item }">
          <div class="gray">{{ item.rank }}</div>
        </template>
        <template v-slot:item.price="{ item }">
          <div>{{ item.price }} €</div>
        </template>
        <template v-slot:item.priceprocentChange24h="{ item }">
          <div :style="getColor(item.priceprocentChange24h)">
            {{ item.priceprocentChange24h }} %
          </div>
        </template>
        <template v-slot:item.volume="{ item }">
          <div>{{ item.volume.toLocaleString("en-US") }}</div>
        </template>
      </v-data-table>
    </v-card>
    <v-dialog v-model="showModal" width="500" v-if="details">
      <template>
        <v-card>
          <v-card-title class="text-h5 grey lighten-2">
            <img :src="details.image.large" />
            <div class="coin">{{ details.name }}</div>
            <div class="coin gray">{{ details.symbol.toUpperCase() }}</div>
          </v-card-title>

          <v-card-text>
            <div v-if="details.genesis_date">
              Created in {{ details.genesis_date }}
            </div>
          </v-card-text>
          <v-card-text>
            <div v-html="details.description.en"></div>
          </v-card-text>
          <v-card-text>
            <div v-if="details.links.homepage[0]"></div>
            <a :href="details.links.homepage[0]" target="_blank">{{
              details.links.homepage[0]
            }}</a>
          </v-card-text>

          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" @click="showModal = false"> EXIT </v-btn>
          </v-card-actions>
        </v-card>
      </template>
    </v-dialog>
    <div class="selectDiv">
      <v-row>
        <v-col cols="12" sm="6" align="center">
          <v-select
            class="selectPerPage"
            v-model="itemsPerPage"
            :items="items"
            label="Item per page:"
            @change="getData"
          ></v-select>
        </v-col>
        <v-col cols="12" sm="6" align="center">
          <v-pagination
            v-model="currentPage"
            :length="100"
            @input="pageChange"
            :total-visible="5"
          >
          </v-pagination>
        </v-col>
      </v-row>
    </div>
  </v-app>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      itemsPerPage: 100,
      items: [10, 20, 50, 100],
      currentPage: 1,
      search: "",
      error: [],
      response: [],
      details: null,
      showModal: false,
      headers: [
        {
          value: "rank",
          text: "#",
        },
        {
          text: "Coin",
          align: "start",
          value: "coin",
        },
        { text: "Price", value: "price" },
        {
          text: "Price Change %",
          value: "priceprocentChange24h",
        },
        { text: "Volume", value: "volume" },
      ],
    };
  },
  methods: {
    // turn on modal
    showDetails() {
      return (this.showModal = true);
    },
    getDetailsCoin(value) {
      //clear deatails about coin
      this.details = null;
      //open modal
      this.showDetails();
      //get API with coin's details
      axios
        .get(`https://api.coingecko.com/api/v3/coins/${value.id}`)
        .then((details) => {
          this.details = details.data;
        })
        .then()
        .catch((e) => {
          this.error.push(e);
        });
    },
    // change color depend on price changing
    getColor(priceprocentChange24h) {
      if (priceprocentChange24h > 0) return "color: green";
      else if (priceprocentChange24h < 0) return "color: red";
      else return " color: black";
    },
    // get api with list of coins
    getData() {
      axios
        .get(
          `https://api.coingecko.com/api/v3/coins/markets?vs_currency=eur&order=market_cap_desc&per_page=${this.itemsPerPage}&page=${this.currentPage}&sparkline=false`
        )
        .then((response) => {
          this.response = response.data.map((o) => ({
            rank: o.market_cap_rank,
            id: o.id,
            //
            image: o.image,
            name: o.name,
            symbol: o.symbol,
            //
            price: o.current_price,
            //
            priceprocentChange24h: o.price_change_percentage_24h,
            //
            volume: o.total_volume,
          }));
        })
        .catch((e) => {
          this.error.push(e);
        });
    },
    pageChange(value) {
      this.currentPage = value;
      this.getData();
    },
  },
  mounted() {
    this.getData();
  },
  computed: {
    //search input filtering
    filteredResponse() {
      let filteredByName = this.response.filter((item) => {
        const arr = this.search.toLowerCase().split(" ");
        return arr.every(
          (el) =>
            item.name.toLowerCase().includes(el) ||
            item.symbol.toLowerCase().includes(el)
        );
      });
      return filteredByName;
    },
  },
};
</script>

<style>
.coin {
  margin: auto 10px;
}
.gray {
  color: gray;
}
img {
  height: 50px;
  width: 50px;
  padding: 5px;
}
.selectPerPage {
  width: 60%;
}
.selectDiv {
  margin: 20px;
}
</style>
