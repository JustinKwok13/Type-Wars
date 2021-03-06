<template>
  <div>
    <vs-card class="online-players-cards-wrapper">
      <template #title>
        <h3>Online Players</h3>
      </template>

      <template #text>
        <transition-group class="online-users" name="list" tag="ul">
          <li
            class="online-user"
            v-for="onlineUser in onlineUsers"
            :key="onlineUser.email"
          >
            <vs-avatar circle size="35" badge badge-color="success">
              <img
                :src="onlineUser.avatar"
                :alt="onlineUser.name + ' avatar image'"
              />
            </vs-avatar>
            <div class="user-details">
              <h4>{{ onlineUser.name }}</h4>
              <!-- <p>{{ user.email }}</p> -->
            </div>

            <div
              class="user-actions-wrapper"
              v-if="onlineUser.email !== user.email"
            >
              <vs-tooltip circle top class="icon-wrapper">
                <GamepadIcon @click="handleChallengeUserClick(onlineUser)" />
                <template #tooltip> Challenge for a game </template>
              </vs-tooltip>
              <!-- <vs-tooltip circle top class="icon-wrapper">
                <SupportIcon />
                <template #tooltip> Send a friend request </template>
              </vs-tooltip> -->
            </div>
          </li>
        </transition-group>
      </template>
    </vs-card>

    <vs-dialog
      width="550px"
      blur
      center
      v-if="showChallengeUserConfirmModal"
      v-model="showChallengeUserConfirmModal"
    >
      <template #header>
        <h4 class="not-margin">Challenge {{ userSelectedForAGame.name }}</h4>
      </template>

      <div class="con-content">
        <div class="dialog-user">
          <vs-avatar circle size="35">
            <img :src="userSelectedForAGame.avatar" alt="User Avatar" />
          </vs-avatar>
          <div>
            <h4>{{ userSelectedForAGame.name }}</h4>
            <!-- <p>some other user statistics</p> -->
          </div>
        </div>
        <vs-input
          type="text"
          label="Challenge Message"
          v-model="challengeMessage"
        >
        </vs-input>
      </div>

      <template #footer>
        <div class="con-footer">
          <vs-button @click="challengeUserForAGame" active>
            Challenge
          </vs-button>
          <vs-button
            @click="showChallengeUserConfirmModal = false"
            dark
            transparent
          >
            Cancel
          </vs-button>
        </div>
      </template>
    </vs-dialog>
  </div>
</template>

<script>
import { mapState } from "vuex";
import GamepadIcon from "@/assets/icons/gamepad.svg";
import SupportIcon from "@/assets/icons/support.svg";

import ChallengeNotificationContent from "@/components/App/ChallengeNotificationContent";

export default {
  components: {
    GamepadIcon,
    SupportIcon,
    ChallengeNotificationContent,
  },
  computed: {
    ...mapState("user", ["user", "onlineUsers", "socketId"]),
  },
  data() {
    return {
      userSelectedForAGame: null,
      showChallengeUserConfirmModal: false,
      challengeMessage: "",
      someoneChallengedYou: false,
      challengeData: null,
    };
  },
  sockets: {
    SOMEONE_CHALLEGED_YOU(challengeData) {
      this.$toast(
        {
          component: ChallengeNotificationContent,
          props: {
            challengeData,
          },
        },
        {
          timeout: false,
          icon: false,
        }
      );
    },
    SOMEONE_ASKED_FOR_A_REMATCH(challengeData) {
      this.$toast(
        {
          component: ChallengeNotificationContent,
          props: {
            challengeData,
            isARematch: true,
          },
        },
        {
          timeout: false,
          icon: false,
        }
      );
    },
    CHALLENGE_DECISION(challengeDecisionData) {
      this.$toast.clear();
      let notificationMessage = "";
      if (challengeDecisionData.decision === "ACCEPT") {
        notificationMessage = `${challengeDecisionData.challengedUser.name} has accepted your challenge.`;
        this.$toast.success(notificationMessage);
        this.handleChallengeAccepted(challengeDecisionData);
      } else {
        notificationMessage = `${challengeDecisionData.challengedUser.name} has rejected your challenge.`;
        this.$toast.info(notificationMessage);
      }
    },
    GAME_IS_STARTING(data) {
      this.$store.commit("game/SET_ACTIVE_GAME_DETAILS", data);
      this.$router.push({ path: "game", query: { roomId: data.roomId } });
    },
  },
  methods: {
    handleChallengeAccepted(challengeDecisionData) {
      this.$socket.emit(
        "CHALLENGER_SIGNAL_TO_START_GAME",
        challengeDecisionData
      );
    },
    handleChallengeUserClick(userSelectedForAGame) {
      this.userSelectedForAGame = userSelectedForAGame;
      this.showChallengeUserConfirmModal = true;
    },
    challengeUserForAGame() {
      const payload = {
        challenger: { ...this.user, socketId: this.socketId },
        challengedUser: this.userSelectedForAGame,
        message: this.challengeMessage,
      };

      this.$socket.emit("SOMEONE_CHALLENGED_FOR_A_GAME", payload);
      this.challengeMessage = "";
      this.userSelectedForAGame = null;
      this.showChallengeUserConfirmModal = false;

      this.$toast.success("Challenge request sent. Awaiting the decision.", {
        timeout: 3000,
      });
    },
  },

  mounted() {},
};
</script>

<style lang="scss">
.vs-card {
  height: 75vh !important;
  overflow-y: auto;
}
.online-players-cards-wrapper {
  .vs-card__title {
    position: sticky;
    top: 2rem;
    left: 0;
    z-index: 20;
    background-color: white;
  }
  .online-users {
    margin-top: 3rem;
    .online-user {
      display: flex;
      align-items: center;
      padding: 0;
      padding: 8px;
      border-radius: 5px;
      margin-bottom: 8px;
      overflow: hidden;

      &:hover {
        background-color: #f8ececbd;

        h4 {
          color: #ff7f8a;
        }

        svg {
          fill: #ff7f8a;
        }

        .user-actions-wrapper {
          visibility: visible;
          transform: translateX(0);
        }
      }

      .vs-avatar-content {
        margin-right: 8px;
      }

      .user-details {
        max-width: 60%;
        margin-right: auto;
      }

      .user-actions-wrapper {
        display: flex;
        visibility: hidden;
        align-items: center;
        transform: translateX(4.5rem);
        transition: all 0.3s ease-in-out;
        .icon-wrapper {
          display: flex;
          align-items: center;
          margin-left: 12px;
          svg {
            width: 24px;
            height: 24px;
            transition: all 0.1s ease-in-out;

            &:hover {
              fill: #2c3e50;
            }
          }
        }
      }
    }
  }
}

.dialog-user {
  display: flex;
  align-items: center;
  margin-top: 8px;
  margin-bottom: 3rem;

  .vs-avatar-content {
    margin-right: 8px;
  }
}
.vs-input-parent .vs-input-content {
  margin-bottom: 1rem;

  label {
    margin-bottom: 8px;
    font-size: 14px;
    font-weight: 400;
  }

  input {
    width: 100%;
    border: 1px solid rgb(163, 161, 161) !important;
    border-radius: 3px !important;
  }
}
</style>
