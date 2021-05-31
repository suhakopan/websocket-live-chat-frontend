<template>
  <div>
    <div id="main-content" class="container">
      <div class="alert alert-danger" role="alert" v-if="errorAlert">
        You should connect to the socket server 
      </div>
      <div class="alert alert-success" role="alert" v-if="successAlert">
        Connected...
      </div>
      <div class="row">
        <div class="col-md-12">
          <form class="form-inline">
            <div class="form-group">
              <label for="connect" class="margin-right-20"
                >WebSocket Connection:</label
              >
              <button
                id="connect"
                class="btn btn-primary margin-right-5"
                type="submit"
                :disabled="connected"
                @click.prevent="connect"
              >
                Connect
              </button>
              <button
                id="disconnect"
                class="btn btn-default"
                type="submit"
                :disabled="!connected"
                @click.prevent="disconnect"
              >
                Disconnect
              </button>
            </div>
          </form>
        </div>
      </div>
      <div class="row margin-top">
        <div class="col-md-6">
          <form class="form-inline">
            <div class="form-group">
              <label for="name" class="margin-right-20">Name</label>
              <input
                type="text"
                id="name"
                class="form-control"
                v-model="inputName"
                placeholder="Your name here..."
              />
            </div>
          </form>
        </div>
        <div class="col-md-6">
          <form class="form-inline">
            <div class="form-group">
              <label for="name" class="margin-right-20">Message</label>
              <input
                type="text"
                id="name"
                class="form-control margin-right-5"
                v-model="inputMessage"
                placeholder="Your message here..."
              />
            </div>
            <button
              id="send"
              class="btn btn-primary"
              type="submit"
              @click.prevent="send"
            >
              Send
            </button>
          </form>
        </div>
      </div>
      <div class="row">
        <div class="col-md-12">
          <table id="conversation" class="table table-striped">
            <thead>
              <tr>
                <th>Chat History</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in receivedMessages" :key="item">
                <td><b>{{ item.from }}</b></td>
                <td>{{ item.text }}</td>
                <td>{{ item.time }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import SockJS from "sockjs-client";
import Stomp from "webstomp-client";
export default {
  name: "Chat",
  data() {
    return {
      receivedMessages: [],
      inputName: null,
      inputMessage: null,
      connected: false,
      stompClient: null,
      errorAlert: false,
      successAlert: false,
    };
  },
  methods: {
    send() {
      if (this.stompClient && this.stompClient.connected) {
        const msg = { from: this.inputName, text: this.inputMessage };
        this.stompClient.send("/chat", JSON.stringify(msg), {});
      } else {
          this.errorAlert = true;
      }
    },
    connect() {
      var socket = new SockJS("http://localhost:8080/chat");
      this.errorAlert = false;
      this.stompClient = Stomp.over(socket);
      this.stompClient.connect(
        {},
        () => {
          this.connected = true;
          this.successAlert = true;
          this.stompClient.subscribe("/topic", (message) => {
            this.receivedMessages.push(JSON.parse(message.body));
          });
        },
        (error) => {
          console.log("Connection error:", error);
          this.connected = false;
        }
      );
    },
    disconnect() {
      if (this.stompClient) {
        this.stompClient.disconnect();
      }
      this.connected = false;
      this.successAlert = false;
    },
  },
};
</script>
<style scoped>
.margin-top {
  margin-top: 50px;
}
.margin-right-20 {
  margin-right: 20px;
}
.margin-right-5 {
  margin-right: 5px;
}
</style>
