<template>
  <q-page-container>
    <q-page class="flex flex-column flex-center">
      <div id="chat-container">
        <q-card class="my-card">
          <q-card-section class="bg-primary text-white">
            <div class="text-h6">Simple Chat App</div>
          </q-card-section>

          <q-separator />

          <q-card-section
            id="chat-messages-container"
            style="max-height: 400px; overflow-y: auto"
          >
            <div class="row q-pa-md justify-center" style="width: 100%">
              <div style="width: 100%">
                <q-chat-message
                  v-for="(item, index) in chatMessages"
                  :key="index"
                  :name="item.user"
                  :avatar="item.avatar"
                  :text="[item.text]"
                  :stamp="item.stamp"
                  text-color="white"
                  :bg-color="item.bgColor"
                  :sent="item.user === this.user"
                />
              </div>
            </div>
            <div class="row">
              <div class="col-11">
                <q-input
                  v-model="text"
                  label="Standard"
                  @keyup.enter="sendMessage"
                />
              </div>
              <div class="col-1">
                <q-btn
                  round
                  color="secondary"
                  icon="send"
                  @click="sendMessage"
                  style="margin-top: 10px"
                />
              </div>
            </div>
          </q-card-section>

          <q-separator />
        </q-card>
      </div>
    </q-page>
  </q-page-container>
</template>

<script>
import { defineComponent } from "vue";
const { DeepstreamClient } = window.DeepstreamClient;
const client = new DeepstreamClient("localhost:6020");
client.login();

export default defineComponent({
  name: "IndexPage",
  data() {
    return {
      init: false,
      client: client,
      text: "",
      chatMessages: [],
      userId: null,
      user: null,
    };
  },
  mounted() {
    if (!this.init) {
      this.init = true;
      this.userId = Math.floor(Math.random() * 5);
      this.user = `user-${this.userId}`;
      this.client.event.subscribe(
        "messages",
        function (value) {
          const message = JSON.parse(value);
          this.chatMessages.push({
            text: message.text,
            user: message.user,
            stamp: message.stamp,
            avatar: message.avatar,
            bgColor: message.user === this.user ? "amber-7" : "primary",
            init: message.init,
          });
          const container = document.getElementById("chat-messages-container");
          container.scrollTop = container.scrollHeight + 400;
        }.bind(this)
      );
    }
    if (this.chatMessages.length === 0) {
      this.sendMessage();
    }
  },
  methods: {
    sendMessage() {
      if (this.text.length === 0) return;
      this.client.event.emit(
        "messages",
        JSON.stringify({
          text: this.text,
          user: this.user,
          stamp: this.fromNow(new Date()),
          avatar: `https://cdn.quasar.dev/img/avatar${this.userId}.jpg`,
          bgColor: "amber-7",
          init: false,
        })
      );
      this.text = "";
    },
    fromNow(date) {
      const SECOND = 1000;
      const MINUTE = 60 * SECOND;
      const HOUR = 60 * MINUTE;
      const DAY = 24 * HOUR;
      const WEEK = 7 * DAY;
      const MONTH = 30 * DAY;
      const YEAR = 365 * DAY;
      const units = [
        {
          max: 30 * SECOND,
          divisor: 1,
          past1: "just now",
          pastN: "just now",
          future1: "just now",
          futureN: "just now",
        },
        {
          max: MINUTE,
          divisor: SECOND,
          past1: "a second ago",
          pastN: "# seconds ago",
          future1: "in a second",
          futureN: "in # seconds",
        },
        {
          max: HOUR,
          divisor: MINUTE,
          past1: "a minute ago",
          pastN: "# minutes ago",
          future1: "in a minute",
          futureN: "in # minutes",
        },
        {
          max: DAY,
          divisor: HOUR,
          past1: "an hour ago",
          pastN: "# hours ago",
          future1: "in an hour",
          futureN: "in # hours",
        },
        {
          max: WEEK,
          divisor: DAY,
          past1: "yesterday",
          pastN: "# days ago",
          future1: "tomorrow",
          futureN: "in # days",
        },
        {
          max: 4 * WEEK,
          divisor: WEEK,
          past1: "last week",
          pastN: "# weeks ago",
          future1: "in a week",
          futureN: "in # weeks",
        },
        {
          max: YEAR,
          divisor: MONTH,
          past1: "last month",
          pastN: "# months ago",
          future1: "in a month",
          futureN: "in # months",
        },
        {
          max: 100 * YEAR,
          divisor: YEAR,
          past1: "last year",
          pastN: "# years ago",
          future1: "in a year",
          futureN: "in # years",
        },
        {
          max: 1000 * YEAR,
          divisor: 100 * YEAR,
          past1: "last century",
          pastN: "# centuries ago",
          future1: "in a century",
          futureN: "in # centuries",
        },
        {
          max: Infinity,
          divisor: 1000 * YEAR,
          past1: "last millennium",
          pastN: "# millennia ago",
          future1: "in a millennium",
          futureN: "in # millennia",
        },
      ];
      const diff =
        Date.now() -
        (typeof date === "object" ? date : new Date(date)).getTime();
      const diffAbs = Math.abs(diff);
      for (const unit of units) {
        if (diffAbs < unit.max) {
          const isFuture = diff < 0;
          const x = Math.round(Math.abs(diff) / unit.divisor);
          if (x <= 1) return isFuture ? unit.future1 : unit.past1;
          return (isFuture ? unit.futureN : unit.pastN).replace("#", x);
        }
      }
    },
  },
});
</script>
<style>
.flex-column {
  flex-direction: column;
}
#chat-container {
  width: 600px;
}
</style>
