<template>
  <Loader v-if="showLoading"></Loader>
  <Wallet v-if="isAuthenticated" />
  <Navigation />
  <div class="alert alert-danger" v-if="noProvider">
    Please install MetaMask! -> More info
    <router-link to="/metamask">here</router-link>
  </div>
  <div>
    <router-view></router-view>
  </div>
</template>

<script>
import Navigation from "./components/Navigation.vue";
import Wallet from "./components/Wallet.vue";
import { mapState, mapGetters, mapActions } from "vuex";
import { defineAsyncComponent } from "vue";

const Loader = defineAsyncComponent(() =>
  import(/* webpackChunkName: "Loader" */ "./components/Loader.vue")
);

export default {
  name: "App",
  computed: {
    ...mapState({
      showLoading: (state) => state.showLoading,
      web3Provider: (state) => state.web3Provider,
    }),
    ...mapGetters([
      "noProvider",
      "notOnlineWeb3",
      "notEnabled",
      "isAuthenticated",
    ]),
  },
  watch: {
    noProvider(curValue, oldValue) {
      if (curValue && curValue !== oldValue) {
        console.log("qua");
        this.web3ProviderInterval();
      } else if (!curValue && curValue !== oldValue) {
        this.web3Provider.on("connect", () => this.checkWeb3Provider());
        this.web3Provider.on("disconnect", () => this.checkWeb3Provider());
        this.web3Provider.on("accountsChanged", () => {
          if (this.isAuthenticated) {
            this.logout();
            this.$router.replace("/login");
          }
          this.enableWeb3Provider();
        });
        this.web3Provider.on("chainChanged", () => this.checkNetworkId());
      }
    },
  },
  components: {
    Navigation,
    Loader,
    Wallet,
  },
  created() {
    const ipfs = this.$ipfs;
    this.web3ProviderInterval();
    this.initIPFS({ ipfs });
  },
  methods: {
    ...mapActions([
      "checkWeb3Provider",
      "enableWeb3Provider",
      "checkNetworkId",
      "initIPFS",
      "logout",
    ]),
    async web3ProviderInterval() {
      await this.checkWeb3Provider();
      if (!this.web3Provider) {
        setTimeout(() => this.web3ProviderInterval(), 5000);
      }
    },
  },
};
</script>

