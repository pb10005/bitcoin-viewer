<template>
  <section>
    <div>
      <section class="hero is-primary">
        <NLink class="button is-primary is-light" to="/">＜ Home</NLink>
        <div class="hero-body">
          <div class="container">
            <h2 class="title center">1BTC ={{ticker.ltp}}JPY</h2>
            <h1 class="subtitle center">Bitcoin</h1>
          </div>
        </div>
      </section>
      <ul class="panel">
        <li class="panel-heading">約定履歴</li>
        <li class="panel-block" v-for="item in executions" :key="item.id">
          <span class="tag is-danger" v-if="item.side === 'SELL'">{{item.side}}</span>
          <span class="tag is-success" v-else>{{item.side}}</span>
          <span>{{item.price}}</span>
          <span>({{item.size}}BTC)</span>
        </li>
      </ul>
    </div>
  </section>
</template>

<script>
import io from "socket.io-client";
export default {
  components: {},
  data() {
    return {
      ticker: {},
      executions: []
    };
  },
  mounted() {
    const socket = io("https://io.lightstream.bitflyer.com", {
      transports: ["websocket"] // specify explicitly
    });
    const publicChannels = [
      "lightning_ticker_BTC_JPY",
      "lightning_executions_BTC_JPY"
    ];
    // connection handling
    socket.on("connect", () => {
      // subscribe to the Public Channels
      for (const ch of publicChannels) {
        socket.emit("subscribe", ch, err => {
          if (err) {
            console.error(ch, "Subscribe Error:", err);
            return;
          }
          console.log(ch, "Subscribed.");
        });
      }
    });

    for (const ch of [...publicChannels]) {
      socket.on(ch, message => {
        switch (ch) {
          case "lightning_ticker_BTC_JPY":
            this.ticker = message;
            break;
          case "lightning_executions_BTC_JPY":
            this.executions = [...message, ...this.executions].slice(0, 20);
            break;
          default:
        }
      });
    }
  },
  methods: {}
};
</script>

<style scoped>
</style>
