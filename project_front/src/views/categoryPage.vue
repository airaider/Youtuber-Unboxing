<template>
  <div>
    <v-card>
      <v-card-title class="justify-center py-6" style="background-color:#ffb468 ; height : 300px">
        <v-icon size="70" color="white" class="mr-2">mdi-widgets-outline</v-icon>
        <span style="text-shadow: 0 0 2px #000;font-size: 2.5em;color:white">CATEGORY</span>
      </v-card-title>
    </v-card>
    <v-container name="container" background-color="transparent" class="pa-1">
      <v-tabs :value="currentCategory" background-color="transparent" grow height="60px">
        <v-tab key="0" @click="onCategoryButtonClicked(0)">
          <div class="categorySet">
            <!-- <v-btn block :aspect-ratio="1/1" @click="onCategoryButtonClicked(0)" class="categorySet"> -->
            <v-icon class="material-icons" color="red" large>mdi-widgets-outline</v-icon>
            <!-- </v-btn> -->
            <div class="categoryName" style="color:black; font-weight: lighter">전체</div>
          </div>
        </v-tab>
        <v-tab
          v-for="(item,index) in categories"
          :key="index+1"
          @click="onCategoryButtonClicked(index+1)"
        >
          <div class="categorySet">
            <v-icon class="material-icons" :color="item.iconColor" large>{{item.icon}}</v-icon>
            <div class="categoryName" style="color:black; font-weight: lighter">{{item.iconName}}</div>
          </div>
        </v-tab>
      </v-tabs>

      <v-tabs-items :value="currentCategory" height="60px">
        <v-tab-item key="0">
          <v-card flat class="pa-3" color="#FAFAFA">
            <v-card-title class="pa-0" style="background-color : white">
              <v-spacer></v-spacer>
              <v-spacer></v-spacer>
              <v-spacer></v-spacer>

              <v-text-field
                v-model="search"
                append-icon="search"
                label="원하는 유튜버를 검색해보세요"
                single-line
                hide-details
                class="mr-5 mb-5"
              ></v-text-field>
            </v-card-title>
            <v-data-table :headers="headers" :items="youtubersPerCategory" :search="search">
              <template v-slot:item.insertCompare="{ item }">
                <v-btn
                  @click="onClikcedinsertCompare(item.yno, item.channelName)"
                  text
                  color="green"
                  :disabled="disabledButtonFlag"
                >
                  <v-icon dark>mdi-login-variant</v-icon>비교담기
                </v-btn>
              </template>

              <!-- 썸네일과 channelName -->
              <template v-slot:item.channelName="{ item }">
                <v-card
                  color="#00000000"
                  flat
                  :to="{path: 'youtuberPage', query: { yno: item.yno }}"
                >
                  <v-row>
                    <v-col cols="2" class="px-0">
                      <v-card color="#00000000" width="50px" flat>
                        <v-responsive :aspect-ratio="1/1">
                          <v-img class="circle" :src="item.thumbnails" flat />
                        </v-responsive>
                      </v-card>
                    </v-col>
                    <v-col cols="10" class="px-0">
                      <v-container fill-height>
                        <v-layout align-center>
                          <v-flex xs12 text-xs-center>
                            <div class="font-weight-light">{{ item.channelName }}</div>
                          </v-flex>
                        </v-layout>
                      </v-container>
                    </v-col>
                  </v-row>
                </v-card>
              </template>
              <template v-slot:item.subscriber="{ item }">
                {{tc(item.subscriber)}}
              </template>
              <template v-slot:item.totalViewCount="{ item }">
                {{tc(item.totalViewCount)}}
              </template>
              <template v-slot:item.influence="{ item }">
                {{item.influence}}점
              </template>
              <template v-slot:item.activity="{ item }">
                {{item.activity}}점
              </template>
              <template v-slot:item.charm="{ item }">
                {{item.charm}}점
              </template>
              <template v-slot:item.grade="{ item }">
                {{
                setGrade(item.grade)
                }}
              </template>

              
            </v-data-table>
          </v-card>
        </v-tab-item>

        <v-tab-item v-for="(item, index) in categories" :key="index + 1">
          <v-card flat class="pa-3" color="#FAFAFA">
            <v-card-title class="pa-0" style="background-color : white">
              <v-spacer></v-spacer>
              <v-spacer></v-spacer>
              <v-spacer></v-spacer>

              <v-text-field
                v-model="search"
                append-icon="search"
                label="원하는 유튜버를 검색해보세요"
                single-line
                hide-details
                class="mr-5 mb-5"
              ></v-text-field>
            </v-card-title>
            <v-data-table
              flat
              :headers="headers"
              :items="youtubersPerCategory"
              class="elevation-1"
              :search="search"
            >
              <template v-slot:item.insertCompare="{ item }">
                <v-btn
                  @click="onClikcedinsertCompare(item.yno, item.channelName)"
                  text
                  color="green"
                  :id="item.yno"
                  :disabled="disabledButtonFlag"
                >
                  <v-icon dark>mdi-login-variant</v-icon>비교담기
                </v-btn>
              </template>

              <!-- 썸네일 -->
              <template v-slot:item.channelName="{ item }">
                <v-card
                  color="#00000000"
                  flat
                  :to="{ path: 'youtuberPage', query: { yno: item.yno } }"
                >
                  <v-row>
                    <v-col cols="2" class="px-0">
                      <v-card color="#00000000" width="50px" flat>
                        <v-responsive :aspect-ratio="1 / 1">
                          <v-img class="circle" :src="item.thumbnails" flat />
                        </v-responsive>
                      </v-card>
                    </v-col>
                    <v-col cols="10" class="px-0">
                      <v-container fill-height>
                        <v-layout align-center>
                          <v-flex xs12 text-xs-center>
                            <div>{{ item.channelName }}</div>
                          </v-flex>
                        </v-layout>
                      </v-container>
                    </v-col>
                  </v-row>
                </v-card>
              </template>

              <template v-slot:item.subscriber="{ item }">
                {{
                tc(item.subscriber)
                }}
              </template>

              <template v-slot:item.grade="{ item }">
                {{
                setGrade(item.grade)
                }}
              </template>
            </v-data-table>
          </v-card>
        </v-tab-item>
      </v-tabs-items>
    </v-container>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import Constant from "../vuex/Constant";
import EventBus from "../components/eventBus";
import tc from "thousands-counter";

export default {
  components: {},
  name: "categoryPage",

  methods: {
    onCategoryButtonClicked(index) {
      localStorage.setItem("currentCategory", index);
      if (index == 0) {
        this.$store.dispatch(Constant.GET_ALLYOUTUBER, {
          failCallback: this.failCallback
        });
      } else {
        this.$store.dispatch(Constant.GET_YOUTUBERS_PER_CATEGORY, {
          failCallback: this.failCallback,
          category: this.findCano()
        });
      }
    },
    findCano: function() {
      return this.categories[localStorage.getItem("currentCategory") - 1].cano;
    },
    onClikcedinsertCompare: function(yno, channelName) {
      EventBus.$emit("insertYoutuber", yno, channelName);
      this.calculateCompareDisabled();
    },
    tc(num) {
      return tc(num);
    },
    setGrade(num) {
      if (num >= 95) {
        return "SS";
      } else if (num >= 90) {
        return "S";
      } else if (num >= 80) {
        return "A";
      } else if (num >= 50) {
        return "B";
      } else if (num >= 20) {
        return "C";
      } else {
        return "D";
      }
    },
    failCallback() {
      window.location.reload();
    },
    calculateCompareDisabled() {
      var output = localStorage.getItem("compareYoutuber");
      var arr = JSON.parse(output);

      if (arr != null && arr.length == 2) {
        this.disabledButtonFlag = true;
      } else {
        this.disabledButtonFlag = false;
      }
    }
  },
  mounted() {
    this.$vuetify.goTo(0);
    console.log(localStorage.getItem("currentCategory"));
    if (localStorage.getItem("currentCategory") == 0) {
      this.$store.dispatch(Constant.GET_ALLYOUTUBER, {
        failCallback: this.failCallback
      });
      return;
    }
    this.$store.dispatch(Constant.GET_YOUTUBERS_PER_CATEGORY, {
      failCallback: this.failCallback,
      category: this.findCano()
    });
  },
  created() {
    EventBus.$on("deleteCompare", () => {
      this.calculateCompareDisabled();
    });
    this.calculateCompareDisabled();
  },

  computed: {
    ...mapGetters(["categories"]),
    ...mapGetters(["youtubersPerCategory"]),
    currentCategory() {
      return Number(localStorage.getItem("currentCategory"));
    }
  },
  watch: {},
  data() {
    return {
      headers: [
        { text: "", value: "", sortable: false, width: "1%" },
        { text: "", value: "channelName", sortable: false, width: "30%" },
        { text: "구독자수", value: "subscriber", align: "center" },
        { text: "총조회수", value: "totalViewCount", align: "center" },
        { text: "영향력", value: "influence", align: "center" },
        { text: "활동력", value: "activity", align: "center" },
        { text: "호감도", value: "charm", align: "center" },
        { text: "등급", value: "grade", align: "center" },
        { text: "", value: "insertCompare", sortable: false, align: "center" }
      ],
      search: "",
      disabledButtonFlag: true
    };
  }
};
</script>

<style scoped>
/* .jb {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.circle {
  border-radius: 50%;
} */
</style>
