<template>
  <v-layout align-center justify-center>
    <v-flex xs12 sm8 md4 lg4>
      <v-card class="elevation-1 pa-3">
        <v-card-text>
          <div class="layout column align-center">
            <h1 class="flex my-4 primary--text">Create Room</h1>
          </div>
          <v-form>
            <v-text-field
              label="Room Name *"
              v-model="Room.RoomName"
            ></v-text-field>
            <v-text-field
              label="User Name *"
              v-model="Room.Username"
            ></v-text-field>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="success" @click="createRoom">Create</v-btn>
        </v-card-actions>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import Util from "@/util";

export default {
  data() {
    return {
      Room: {
        RoomName: "",
        Username: "",
      },
    };
  },
  methods: {
    createRoom() {
      const room = {
        Room: {
          RoomId: Util.createUniqueString(),
          RoomName: this.Room.RoomName,
          EstimateResult: 0,
          Action: "hide",
        },
        User: {
          UserId: Util.createUniqueString(),
          Role: "admin",
          Status: true,
          Username: this.Room.Username,
          StoryPoint: 0,
        },
      };
      this.$fire.database
        .ref(room.Room.RoomId + "/Room")
        .set(room.Room)
        .then((response) => {
          this.$fire.database
            .ref(room.Room.RoomId + "/User/" + room.User.UserId)
            .set(room.User);
          const roomData = {
            RoomId: room.Room.RoomId,
            UserId: room.User.UserId,
            Username: room.User.Username,
            Role: "admin",
          };
          let roomDataStorage = JSON.parse(localStorage.getItem("roomData")) || [];
          if (roomDataStorage && roomDataStorage.length > 0) {
            roomDataStorage.push(roomData);
          } else {
            roomDataStorage = [roomData];
          }
          localStorage.setItem("roomData", JSON.stringify(roomDataStorage));
          this.$nuxt.$emit("roomInfo", {
            RoomId: room.Room.RoomId,
            RoomName: room.Room.RoomName,
          });
          this.$router.push({
            path: `/room/${room.Room.RoomId}`,
          });
          // window.location = "/room/" + room.Room.RoomId;
        });
    },
  },
  async mounted() {},
};
</script>
<style scoped lang="css">
#home {
  height: 50%;
  width: 100%;
  position: absolute;
  top: 0;
  left: 0;
  content: "";
  z-index: 0;
}
</style>