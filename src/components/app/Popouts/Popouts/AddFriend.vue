<template>
  <div class="dark-background add-friend-popout" @mousedown="backgroundClick">
    <div class="add-friend">
      <div class="header">
        <div class="icon material-icons">person_add</div>
        <div class="name">{{$t('add-friend')}}</div>
      </div>
      <div class="content">
        <div class="container">
          <div class="description">
            {{$t("add-friend-message")}}
          </div>
          <input
            class="tag"
            v-model="input"
            type="text"
            placeholder="pancake:time"
          />
          <input
            class="button"
            @click="addButton"
            type="button"
            :value="requestSent ? $t('adding') : $t('add-friend')"
          />
        </div>
      </div>
      <div class="alerts" :class="{ success, error }" v-if="error || success">
        {{ error || success || "" }}
      </div>
    </div>
  </div>
</template>

<script>
import RelationshipService from "@/services/RelationshipService.js";

export default {
  data() {
    return {
      input: "",
      requestSent: false,
      error: null,
      success: null
    };
  },
  computed: {},
  methods: {
    closeMenu() {
      this.$store.dispatch("setAllPopout", {
        show: false,
        type: "ADD_FRIEND"
      });
    },
    backgroundClick(e) {
      if (e.target.classList.contains("dark-background")) {
        this.closeMenu();
      }
    },
    async addButton() {
      if (this.requestSent) return;
      this.error = null;
      this.success = null;
      this.requestSent = true;

      const split = this.input.trim().split(":");
      // validation
      if (
        split.length < 2 ||
        split.length > 2 ||
        split[1] === "" ||
        split[1].length !== 4
      ) {
        this.requestSent = false;
        this.error = "Invalid username or tag.";
        return;
      }
      const username = split[0];
      const tag = split[1];

      const { ok, error, result } = await RelationshipService.post({
        username,
        tag
      });
      this.requestSent = false;
      if (ok) {
        this.success = result.data.message;
      } else {
        if (error.response === undefined) {
          this.error = "Can't connect to server.";
          return;
        }
        this.error = error.response.data.errors[0].msg;
      }
    }
  }
};
</script>

<style scoped lang="scss">
.dark-background {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.541);
  z-index: 111111;
  display: flex;
}

.add-friend {
  height: 300px;
  width: 400px;
  backdrop-filter: blur(10px);
  display: flex;
  flex-direction: column;
  position: relative;
  margin: auto;
  overflow: hidden;
  box-shadow: 0px 0px 20px 5px #151515bd;
  background: linear-gradient(
    to bottom,
    rgba(0, 87, 153, 0.8) 0,
    rgba(0, 118, 209, 0.8)
  );
  border-radius: 4px;
  backdrop-filter: blur(5px);
}
.header {
  display: flex;
  flex-shrink: 0;
  background: rgba(0, 0, 0, 0.4);
  color: white;
  height: 50px;
  align-content: center;
  align-items: center;
  justify-content: center;
  .icon {
    margin-right: 5px;
  }
}

.content {
  display: flex;
  position: relative;
  flex-direction: column;
  height: 100%;
  width: 100%;
  color: rgb(255, 255, 255);
}
.container {
  display: flex;
  flex-direction: column;
  margin: auto;
  align-items: center;
  align-content: center;
  margin-top: 40px;
}

.description {
  margin-bottom: 30px;
}
.button {
  align-self: center;
  color: white;
  width: initial;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 4px;
  cursor: pointer;
  &:hover {
    background: rgba(0, 0, 0, 0.4);
  }
}
input {
  border-radius: 4px;
}

.alerts {
  display: flex;
  flex-direction: column;
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 30px;
  color: white;
  align-items: center;
  align-content: center;
  justify-content: center;
  &.success {
    background: #008a00;
  }
  &.error {
    background: #8a0000;
  }
}
</style>
