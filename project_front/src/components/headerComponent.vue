<template>
  <v-app-bar id="app_bar_tt" color="#222222" fixed elevate-on-scroll scroll-threshold="500" app>
    <v-col cols="2" class="pa-0 pt-2 pb-1">
      <v-img
        :src="require('@/assets/logo4.png')"
        contain
        height="60"
        width="185"
        max-width="185"
        @click="gotoHome"
        style="cursor:pointer"
      />
    </v-col>
    <v-col cols="6" class="ml-3">
      <v-row>
        <v-btn
          dark
          text
          class="headerMenu"
          style="font-size: 20px;"
          @click="gotoPage('/howToPage')"
        >
          <span style="text-shadow: 0 0 2px #000;">MANUAL</span>
        </v-btn>
        <v-btn
          dark
          text
          class="headerMenu"
          style="font-size: 20px;"
          @click="gotoPage('/categoryPage')"
        >
          <span style="text-shadow: 0 0 2px #000;">CATEGORY</span>
        </v-btn>
        <v-btn dark text class="headerMenu" style="font-size: 20px;" @click="gotoPage('/rankPage')">
          <span style="text-shadow: 0 0 2px #000;">RANK</span>
        </v-btn>
        <input-component v-if="$session.get('token') != undefined" position="header"></input-component>
      </v-row>
    </v-col>

    <v-col cols="4" class="pa-0" align="right">
      <v-flex>
        <v-layout align-center justify-end row fill-height>
          <v-autocomplete
            :items="searchItems"
            :search-input.sync="inputKeyword"
            hide-details
            item-text="channelName"
            item-value="channelName"
            @keyup.enter="search"
            ref="keyword"
            id="keyword"
            label="유튜버를 검색해보세요"
            style="max-width: 230px"
            solo-inverted
            background-color="#bbbbbb"
            flat
            v-if="$route.path != '/'"
            align="center"
            class="justify-center pr-2"
            dense
          >
            <template v-slot:no-data>
              <v-list-item>
                <v-list-item-title>검색 결과가 없습니다!</v-list-item-title>
              </v-list-item>
            </template>

            <template v-slot:item="{ item }">
              <div v-if="item.yno == -1" class="ma-0 pa-0"></div>
              <v-list-item-avatar
                v-if="item.yno != -1"
                color="red"
                class="headline font-weight-light white--text"
                @click="gotoYoutuberPage(item.yno)"
              >
                <img :src="item.thumbnails" alt="John" />
              </v-list-item-avatar>
              <v-list-item-content
                v-if="item.yno != -1"
                style="width: 100px"
                @click="gotoYoutuberPage(item.yno)"
              >
                <v-list-item-title v-text="item.channelName"></v-list-item-title>
                <v-list-item-subtitle>구독자 : {{ tc(item.subscriber) }}</v-list-item-subtitle>
              </v-list-item-content>
            </template>
          </v-autocomplete>

          <!-- 카카오로그인 -->
          <v-btn
            @click="login()"
            class="btnFont mr-5 pl-2 pr-2"
            color="#F8E211"
            v-if="loginStatus == false"
          >
            <v-img :src="require('@/assets/kakaologo.png')" class="mr-2 ml-1" width="18px"></v-img>로그인
          </v-btn>
          <v-dialog v-if="loginStatus" v-model="dialog" persistent max-width="600px">
            <template v-slot:activator="{ on }">
              <v-btn class="btnFont mr-5 pl-2 pr-2" color="#F8E211" v-on="on">
                <v-img :src="require('@/assets/kakaologo.png')" class="mr-2 ml-1" width="18px"></v-img>로그아웃
              </v-btn>
              <v-btn
                class="btnFont ml-3 mr-3 pl-2 pr-2"
                color="#F8E211"
                @click="gotoPage('/memberPage')"
              >
                <v-icon left class="ml-1">mdi-account</v-icon>회원정보
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">
                  <v-icon class="mr-2" color="red">mdi-youtube</v-icon>LOGOUT
                </span>
              </v-card-title>
              <v-card-text>
                <v-container>정말로 로그아웃 하겠습니까?</v-container>
                <v-img :src="require('@/assets/emoji.png')" max-width="50" max-height="50"></v-img>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="primary" rounded class="mr-2" text @click="dialog = false">
                  No
                  <v-icon right>mdi-emoticon-cool-outline</v-icon>
                </v-btn>
                <v-btn dark color="red" rounded text @click="logout()">
                  Yes
                  <v-icon right>mdi-logout</v-icon>
                </v-btn>
                <v-text v-if="check()"></v-text>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-layout>
      </v-flex>
      <!-- 검색 -->
    </v-col>
  </v-app-bar>
</template>

<script>
import inputComponent from "./inputComponent";
import { mapGetters } from "vuex";
import http from "../vuex/http-common";
import tc from "thousands-counter";
// import axios from "axios";
export default {
  components: {
    inputComponent
  },
  computed: {
    ...mapGetters(["links"]),
    isIdle() {
      if (this.$session.get("token")) {
        if (this.$store.state.idleVue.isIdle) {
          alert("자동 로그아웃 되었습니다");
          console.log("IDLE");
          this.logout();
        }
      }
      return this.$store.state.idleVue.isIdle;
    }
  },
  mounted() {
    if (!this.$session.exists()) {
      this.loginStatus = false;
      console.log("no session");
    } else {
      if (this.$session.get("token")) {
        this.loginStatus = true;
      }
    }
    var token = this.$route.query.access_Token;
    if (token != undefined) {
      this.$session.start();
      this.$session.set("token", token);
      this.loginStatus = true;
      this.$router.push("/");
    }
  },
  created() {
    this.check;
  },
  methods: {
    gotoPage(address) {
      if(this.$route.path == address){
        window.location.reload()
        return ;
      }
      if(address == '/categoryPage'){
        var tmp = localStorage.getItem("compareYoutuber")
        if(tmp == null)
          localStorage.setItem("currentCategory",0);
      }
      this.$router.push(address);
    },
    login() {
      window.location.href =
        "https://accounts.kakao.com/login?continue=https%3A%2F%2Fkauth.kakao.com%2Foauth%2Fauthorize%3Fclient_id%3Dacdd77956bf757e4be43817374c35738%26redirect_uri%3Dhttp%3A%2F%2F15.165.77.1%3A8080%2FSpringBoot%2Flogin%26response_type%3Dcode";
    },
    logout() {
      let a;

      let result = new Promise((resolve, reject) => {
        http
          .get("/logout/" + this.$session.get("token"))
          .then(response => {
            a = response.data.data.responseCode;
            if (a == "200") {
              this.$session.destroy();
              console.log(this.$route.query);
              if (this.$route.path == "/memberPage") {
                this.$router.push("/");
              }
              window.location.reload();
            }
            resolve(response);
          })
          .catch(err => {
            reject(err);
          });
      });
      console.log(result);
      this.$session.destroy();
      if (this.$route.path == "/memberPage") {
        this.$router.push("/");
      }
      window.location.reload();
    },
    check() {
      if (this.$session.get("token")) {
        http
          .get("/user/search/" + this.$session.get("token"))
          .then(response => {
            if (response.data.data == null) {
              alert("자동 로그아웃 되었습니다");
              this.$session.destroy();
              if (this.$route.path == "/memberPage") {
                this.$router.push("/");
              }
              window.location.reload();
            }
          })
          .catch(error => {
            console.log(error);
          });
      }
    },
    isDisabled() {
      return true;
    },
    gotoHome(e) {
      e.stopPropagation();
      if (this.$route.path === "/") {
        this.$vuetify.goTo(0);
        //window.location.reload()
        return;
      }
      this.$router.push("/", () => {});
      return;
    },
    search: function() {
      console.log(document.getElementById("keyword").value);
      this.$router.push(
        {
          path: "/searchPage",
          query: { word: document.getElementById("keyword").value }
        },
        () => {}
      );
      document.getElementById("keyword").vaule = "";
    },
    // onScroll() {
    //   var scroll = window.pageYOffset;
    //   var value = "#333333";
    //   var position = 250;
    //   if (scroll >= position) {
    //     value += "";
    //   } else {
    //     var tmp = Math.round((scroll / position) * parseInt("", 16)).toString(
    //       16
    //     );
    //     if (tmp.length == 1) {
    //       value += "0" + tmp;
    //     } else {
    //       value += tmp;
    //     }
    //   }

    //   this.headerColor = value;
    // },
    tc(num) {
      return tc(num);
    },
    gotoYoutuberPage(yno) {
      this.$router.push({ path: "youtuberPage", query: { yno: yno } });
    }
  },
  data() {
    return {
      loginStatus: false,
      searchWord: "",
      dialog: false,
      headerColor: "transparent",
      searchItems: [],
      inputKeyword: null
    };
  },
  creted() {
    this.headerColor = "#00000000";
  },
  watch: {
    inputKeyword() {
      // Items have already been loaded
      if (this.searchItems.length > 10) return;

      // Lazily load input items
      http
        .get("/youtuber/all")
        .then(response => {
          this.searchItems = response.data.data;
          var tmp = { yno: -1, channelName: [] };
          for (let index = 0; index < response.data.data.length; index++) {
            tmp["channelName"] += response.data.data[index].channelName;
          }
          this.searchItems.push(tmp);
        })
        .catch(err => {
          console.log(err);
        });
    }
  }
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css?family=Do+Hyeon|Nanum+Gothic|Noto+Sans+KR&display=swap");
#header {
  background-color: #999999;
  position: absolute;
}

.v-toolbar__content,
.v-toolbar__extension {
  background-color: blue;
  color: blue;
}

.app_bar_tt {
  background-color: blue;
}

.disable-events {
  pointer-events: none;
}
.btnFont {
  font-size: 1em;
  font-family: "Noto Sans KR", sans-serif;
}
.headerMenu {
  margin-top: auto;
  margin-bottom: auto;
  margin-right: 20px;
}
</style>
