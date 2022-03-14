<template>
  <v-row>
    <v-col cols="12" sm="8" v-if="Object.keys(userInfo).length !== 0">
      <v-sheet rounded="lg">
        <v-card raised>
          <v-card-title>
            <v-row>
              <v-col cols="12" sm="8">
                <h3>Submit estimate</h3>
              </v-col>
              <v-col cols="12" sm="4">
                <v-slider
                  v-model="limitNumber"
                  :thumb-size="24"
                  min="50"
                  max="200"
                  step="50"
                  ticks="always"
                  tick-size="4"
                  thumb-label="always"
                ></v-slider>
              </v-col>
            </v-row>
          </v-card-title>
          <v-card-text>
            <v-row>
              <v-col v-for="number in limitNumber" :key="number" cols="auto">
                <v-hover>
                  <template v-slot:default="{ hover }">
                    <v-card
                      height="100"
                      width="100"
                      :color="
                        active && numberActive == number
                          ? '#0D47A1'
                          : 'light-blue lighten-2'
                      "
                      rounded="xl"
                      @click="chooseEstimate(number)"
                    >
                      <v-row class="fill-height" align="center" justify="center"
                        ><h2>{{ number }}</h2></v-row
                      >
                      <v-fade-transition>
                        <v-overlay v-if="hover" absolute color="#0D47A1">
                          <!-- <v-btn>See more info</v-btn> -->
                        </v-overlay>
                      </v-fade-transition>
                    </v-card>
                  </template>
                </v-hover>
              </v-col>
            </v-row>
          </v-card-text>
          <v-card-actions> </v-card-actions>
        </v-card>
      </v-sheet>
    </v-col>

    <v-col cols="12" sm="4" v-if="Object.keys(userInfo).length !== 0">
      <v-sheet rounded="lg">
        <v-card raised>
          <div class="display-1 text-center">{{ roomDB.RoomName }}</div>

          <!-- <v-divider class="my-2"></v-divider> -->

          <v-card-title>
            <h3>Results</h3>
            <v-btn
              color="primary"
              class="ma-4"
              v-show="roomDB.Action == 'show' && userInfo.Role == 'admin'"
              @click="deleteEstimates"
            >
              Delete Estimates
            </v-btn>
            <v-spacer />
            <v-btn
              color="blue-grey"
              class="ma-2"
              v-if="userInfo.Role == 'admin'"
              @click="roomAction('hide')"
              >Hide</v-btn
            >
            <v-btn
              color="primary"
              v-if="userInfo.Role == 'admin'"
              @click="roomAction('show')"
            >
              Show
            </v-btn>
          </v-card-title>
          <v-card-text>
            <v-row>
              <v-col cols="8" sm="8">
                <h3>Name</h3>
              </v-col>
              <v-col cols="4" sm="4">
                <h3>Story Points</h3>
              </v-col>
            </v-row>
            <v-list color="transparent">
              <v-divider class="my-2"></v-divider>
              <v-list-item v-for="item in usersDB" :key="item.UserId" link>
                <v-list-item-content>
                  <v-list-item-title>
                    <v-badge
                      :color="item.Status == true ? 'green' : 'pink'"
                      dot
                    >
                      {{ item.Username }}
                    </v-badge>
                    <v-icon v-if="item.Role == 'admin'">mdi-account</v-icon>
                  </v-list-item-title>
                </v-list-item-content>
                <v-list-item-action>
                  <v-btn
                    icon
                    v-show="roomDB.Action == 'hide' && userInfo.Role != 'admin'"
                  >
                    <v-icon color="grey lighten-1">mdi-information</v-icon>
                  </v-btn>
                  <v-list-item-action-text
                    v-show="roomDB.Action == 'show' || userInfo.Role == 'admin'"
                    ><h2>
                      {{ item.StoryPoint }}
                    </h2></v-list-item-action-text
                  >
                </v-list-item-action>
              </v-list-item>
              <v-divider class="my-2"></v-divider>

              <v-list-item link color="grey lighten-4">
                <v-list-item-content>
                  <v-list-item-title> Average </v-list-item-title>
                </v-list-item-content>
                <v-list-item-action>
                  <v-btn
                    icon
                    v-show="roomDB.Action == 'hide' && userInfo.Role != 'admin'"
                  >
                    <v-icon color="grey lighten-1">mdi-information</v-icon>
                  </v-btn>
                  <v-list-item-action-text
                    v-show="roomDB.Action == 'show' || userInfo.Role == 'admin'"
                    ><h2>
                      {{ roomDB.EstimateResult }}
                    </h2></v-list-item-action-text
                  >
                </v-list-item-action>
              </v-list-item>
            </v-list>
          </v-card-text>
          <v-card-actions> </v-card-actions>
        </v-card>
      </v-sheet>
    </v-col>
    <v-dialog v-model="dialog" width="500" persistent>
      <v-card>
        <v-card-title class="text-h5"> Enter Room </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  v-model="usernameModel"
                  label="Display Name*"
                ></v-text-field>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="enterRoom" :disabled="!usernameModel">
            Enter Room
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
import Util from "@/util";

export default {
  data: () => ({
    active: false,
    numberActive: 0,
    limitNumber: 50,
    roomId: "",
    userId: "",
    userRole: "",
    userName: "",
    usersDB: [],
    roomDB: {},
    dialog: true,
    usernameModel: "",
    connectionState: false,
    myConnectionsRef: null,
    connectedRef: null,
    userInfo: {},
  }),
  created() {},
  mounted() {
    //mandatory init data then call method
    this.roomId = this.$route.params.id;
    this.myConnectionsRef = this.$fire.database.ref(this.roomId + "/User");
    this.connectedRef = this.$fire.database.ref(".info/connected");

    this.checkUserInfoBeforeAccess();
    this.onDBCurrentUser();
    this.onDBUsers();
    this.onDBRoom();
    this.onPresence();
  },
  computed: {},
  watch: {},
  methods: {
    checkUserInfoBeforeAccess() {
      const roomData = JSON.parse(localStorage.getItem("roomData")) || [];
      const checkUser = roomData.find((x) => x.RoomId == this.roomId);
      if (checkUser) {
        this.userId = checkUser.UserId;
        this.userRole = checkUser.Role;
        this.userName = checkUser.Username;
        this.dialog = false;
      } else {
        this.dialog = true;
      }
    },
    onDBCurrentUser() {
      this.$fire.database
        .ref(this.roomId + "/User/" + this.userId)
        .on("value", (snapshot) => {
          if (snapshot.exists()) {
            this.userInfo = snapshot.val();
          }
        });
    },
    onDBUsers() {
      this.myConnectionsRef.on("value", (snapshot) => {
        if (snapshot.exists()) {
          this.usersDB = snapshot.val();
        }
      });
    },
    onDBRoom() {
      this.$fire.database.ref(this.roomId + "/Room").on("value", (snapshot) => {
        if (snapshot.exists()) {
          this.roomDB = snapshot.val();
        }
      });
    },
    onPresence() {
      if (!this.dialog) {
        this.connectedRef.on("value", (snap) => {
          if (snap.val() === true) {
            // We're connected (or reconnected)! Do anything here that should happen only if online (or on reconnect)
            let con = this.myConnectionsRef;
            // When I disconnect, remove this device
            con.child(this.userId).onDisconnect().set({
              Role: this.userRole,
              Status: false,
              StoryPoint: 0,
              UserId: this.userId,
              Username: this.userName,
            });
            // Add this device to my connections list
            // this value could contain info about the device or a timestamp too
            con.child(this.userId + "/Status").set(true);
          }
        });
      }
    },
    enterRoom() {
      // Add user
      const user = {
        UserId: Util.createUniqueString(),
        Role: "user",
        Status: true,
        Username: this.usernameModel,
        StoryPoint: 0,
      };
      this.$fire.database
        .ref(this.roomId + "/User/" + user.UserId)
        .set(user)
        .then((response) => {
          let roomDataStorage =
            JSON.parse(localStorage.getItem("roomData")) || [];
          const roomData = {
            RoomId: this.roomId,
            UserId: user.UserId,
            Role: user.Role,
            Username: user.Username,
          };
          if (roomDataStorage && roomDataStorage.length > 0) {
            roomDataStorage.push(roomData);
          } else {
            roomDataStorage = [roomData];
          }
          localStorage.setItem("roomData", JSON.stringify(roomDataStorage));
          window.location.reload();
        });
    },
    chooseEstimate(number) {
      if (number == this.numberActive || !this.active) {
        this.active = !this.active;
      }
      this.numberActive = this.active ? number : 0;
      this.myConnectionsRef
        .child(this.userId + "/StoryPoint")
        .set(this.numberActive);
      this.averageCalc();
    },
    averageCalc() {
      let sum = 0;
      let usersArr = Object.entries(this.usersDB).map((arr) => ({
        id: arr[0],
        value: arr[1].StoryPoint,
        status: arr[1].Status,
      }));
      usersArr = usersArr.filter((x) => x.status && x.value != 0);

      if (usersArr && usersArr.length > 0) {
        sum =
          usersArr.reduce(function (prev, cur) {
            return prev + cur.value;
          }, 0) / usersArr.length;
      }
      this.$fire.database.ref(this.roomId + "/Room/EstimateResult").set(sum);
    },
    roomAction(type) {
      this.$fire.database.ref(this.roomId + "/Room/Action").set(type);
    },
    deleteEstimates() {
      this.active = false;
      this.numberActive = 0;
      let usersArr = Object.entries(this.usersDB).map((arr) => ({
        UserId: arr[0],
        Role: arr[1].Role,
        Status: arr[1].Status,
        Username: arr[1].Username,
        StoryPoint: arr[1].StoryPoint,
      }));
      usersArr.forEach((x) =>
        this.$fire.database
          .ref(this.roomId + "/User/" + x.UserId + "/StoryPoint")
          .set(0)
      );
      this.$fire.database.ref(this.roomId + "/Room/Action").set("hide");
      this.$fire.database.ref(this.roomId + "/Room/EstimateResult").set(0);
    },
  },
};
</script>
<style scoped lang="css">
</style>