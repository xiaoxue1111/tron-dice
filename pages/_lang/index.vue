<template>
  <div class="container">
    <div v-if="platform == 'pc'">
      <!-- pc端布局 -->
      <my-nav :languageGroup="languageGroup" />
      <Loading :isLoading="showLoading"/>
      <div class="main">
        <el-row>
          <el-col :span="7">
              <rank></rank> 
          </el-col>
          <el-col :span="10">

          </el-col>
          <el-col :span="7">

          </el-col>
        </el-row>
        <!-- <div class="cell">
          <Select />
        </div>
        <div class="cell">
          <div class="col col-1">
            <Rank :contractAddress="contractAddress" />
            <other-prize />
          </div>
          <div class="col col-2">
            <Play />
          </div>
        </div>
        <div class="cell">
          <Result :contractAddress="contractAddress" />
        </div> -->
      </div>
      <!--<Footer />-->
    </div>
    <div v-if="platform == 'mobile'">
      <!-- 移动端布局 -->
        移动端
        <!-- <vip></vip> -->
        <invite></invite>
        
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
import MyNav from "~/components/MyNav.vue";
import Select from "~/components/Select.vue";
import Rank from "~/components/Rank.vue";
import OtherPrize from "~/components/OtherPrize.vue";
import Play from "~/components/Play.vue";
import Result from "~/components/Result.vue";
import Loading from "~/components/loading.vue";
import { getBalance, noDebug, getWeeklyRank } from "~/assets/js/common";

import vip from "~/components/Vip";
import invite from "~/components/Invite";

let contractAddress = ""; //测试网
let activityAddress = "TSYuKXyV6pPcxcMJfaqZzt4KUBtncPPPC5";
export default {
  components: {
    MyNav,
    Loading,
    Select,
    Rank,
    OtherPrize,
    Play,
    Result,
    vip,
    invite
  },
  async asyncData() {
    return {};
  },
  data() {
    return {
      tronWeb: null,
      rollBtnDisabled: false,
      lastRollNum: null,
      contractInstance: null,
      contractAddress: "",
      languageGroup: [
        { lng: "en", txt: "English" },
        { lng: "ch", txt: "Chinese" }
      ],
      platform: "pc"
    };
  },
  computed: {
    ...mapState(["showLoading"])
  },
  async mounted() {
    this.platform = this.checkPlatform();
    noDebug();
    this.limit = this.$store.state.limit;
    let isLoadTronWeb = await this.isHasTronWeb();
    if (isLoadTronWeb) {
      await this.timeout(500);
      this.tronWeb = window.tronWeb;
      if (!this.checkLogin()) return;
      this.checkEnv();
      this.$store.commit("SET_ADDRESS", this.tronWeb.defaultAddress);
      const balance = await getBalance(this.$store.state.address.hex);
      this.$store.commit("SET_BALANCE", this.tronWeb.fromSun(balance));
      const contractInstance = await this.tronWeb
        .contract()
        .at(contractAddress);
      this.contractInstance = contractInstance;
      this.$store.commit("SET_CONTRACT_INSTANCE", contractInstance);
      //const myBets = localStorage.my ? JSON.parse(localStorage.my): [];
      //this.$store.commit('SET_MY_BETS',myBets);
    } else {
      this.checkLogin();
    }
  },
  methods: {
    isHasTronWeb() {
      this.isLoading = true;
      let isTronWebLoaded = false;
      return new Promise(resolve => {
        let oTimer = setInterval(() => {
          if (window.tronWeb) {
            this.$store.commit("SET_SHOW_LOADING", false);
            isTronWebLoaded = true;
            clearInterval(oTimer);
            resolve(isTronWebLoaded);
          } else {
            setTimeout(() => {
              this.$store.commit("SET_SHOW_LOADING", false);
              isTronWebLoaded = false;
              resolve(isTronWebLoaded);
              clearInterval(oTimer);
            }, 3000);
          }
        }, 500);
      });
    },
    checkLogin() {
      if (!window.tronWeb && !window.tronWeb.defaultAddress.base58) {
        this.$store.commit("SET_DIALOG_LOGIN", true);
        return false;
      } else {
        return true;
      }
    },
    checkEnv() {
      if (window.tronWeb.eventServer == "https://api.shasta.trongrid.io") {
        contractAddress = "TWhEuRncFxDfBxQcSyf96aEqWvYUduUTKP";
        activityAddress = "TSYuKXyV6pPcxcMJfaqZzt4KUBtncPPPC5";
      } else {
        contractAddress = "TDVJZ53EowTzWnyknSeDqRGSJjoibExZWh";
        activityAddress = "";
      }
      this.$store.commit("SET_CONTRACT_ADDRESS", contractAddress);
      this.$store.commit("SET_ACTIVITYADDRESS", activityAddress);

      console.log(123, this.$store.state.activityAddress);
    },
    timeout(ms) {
      return new Promise(resolve => {
        setTimeout(resolve, ms);
      });
    },
    checkPlatform() {
      var regex_match = /(nokia|iphone|android|motorola|^mot-|softbank|foma|docomo|kddi|up.browser|up.link|htc|dopod|blazer|netfront|helio|hosin|huawei|novarra|CoolPad|webos|techfaith|palmsource|blackberry|alcatel|amoi|ktouch|nexian|samsung|^sam-|s[cg]h|^lge|ericsson|philips|sagem|wellcom|bunjalloo|maui|symbian|smartphone|midp|wap|phone|windows ce|iemobile|^spice|^bird|^zte-|longcos|pantech|gionee|^sie-|portalmmm|jigs browser|hiptop|^benq|haier|^lct|operas*mobi|opera*mini|320x320|240x320|176x220)/i;
      var u = navigator.userAgent;
      if (null == u) {
        return "mobile";
      }
      var result = regex_match.exec(u);

      if (null == result) {
        return "pc";
      } else {
        return "mobile";
      }
    }
  }
};
</script>

<style lang="scss">
.container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background: rgba(48, 51, 87, 0.9);
  color: #fff;
  a {
    color: #b3a6ff;
  }
  .main {
    width: 90%;
    max-width: 12rem;

    flex: 1;
    margin: auto;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    & > .cell {
      &:nth-child(2) {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        .col {
          width: 6.2rem;
          height: 6.6rem;
        }
        .col.col-1 {
          display: flex;
          flex-direction: column;
          justify-content: space-between;
        }
        .col.col-2 {
          background-image: linear-gradient(
              142deg,
              #28297c 0%,
              #21236e 50%,
              #191c60 100%
            ),
            linear-gradient(#242572, #242572);
          background-blend-mode: normal, normal;
          border-radius: 10px;
          border: solid 1px #64e1f5;
        }
      }
      &:nth-child(3) {
        margin-top: 0.35rem;
      }
    }
  }
}
</style>
