<template>
  <v-dialog v-model="forgetFlag" :fullscreen="isMobile" max-width="460">
    <v-card class="login-container" style="border-radius:4px">
      <v-icon class="float-right" @click="forgetFlag = false">
        mdi-close
      </v-icon>
      <div class="login-wrapper">
        <!-- Username -->
        <v-text-field
          v-model="username"
          label="Mailbox number"
          placeholder="Please enter your mailbox number"
          clearable
          @keyup.enter="forget"
        />
        <!-- Verification code -->
        <div class="mt-7 send-wrapper">
          <v-text-field
            maxlength="6"
            v-model="code"
            label="Verification Code"
            placeholder="Please enter the 6-digit verification code"
            @keyup.enter="forget"
          />
          <v-btn :disabled="flag" text small @click="sendCode">
            {{ codeMsg }}
          </v-btn>
        </div>
        <!-- Password -->
        <v-text-field
          v-model="password"
          class="mt-7"
          label="Password"
          placeholder="Please enter your password"
          @keyup.enter="forget"
          :append-icon="show?'mdi-eye':'mdi-eye-off'"
          :type="show?'text':'password'"
          @click:append="show = !show"
        />
        <!-- button -->
        <v-btn
          class="mt-7"
          block
          color="green"
          style="color:#fff"
          @click="forget"
        >
          Confirm
        </v-btn>
        <!-- Login -->
        <div class="mt-10 login-tip">
          Already have an account? <span @click="openLogin">Log in</span>
        </div>
      </div>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  data: function() {
    return {
      username: "",
      code: "",
      password: "",
      flag: true,
      codeMsg: "Send",
      time: 60,
      show: false
    };
  },
  methods: {
    openLogin() {
      this.$store.state.forgetFlag = false;
      this.$store.state.loginFlag = true;
    },
    sendCode() {
      const that = this;
      // eslint-disable-next-line no-undef
      var captcha = new TencentCaptcha(this.config.TENCENT_CAPTCHA, function(res) {
        if (res.ret === 0) {
          //send email
          that.countDown();
          that.axios
            .get("/api/users/code", {
              params: {username: that.username}
            })
            .then(({ data }) => {
              if (data.flag) {
                that.$toast({ type: "success", message: data.message });
              } else {
                that.$toast({ type: "error", message: data.message });
              }
            });
        }
      });
      // Display verification code
      captcha.show();
    },
    countDown() {
      this.flag = true;
      this.timer = setInterval(() => {
        this.time--;
        this.codeMsg = this.time + "s";
        if (this.time <= 0) {
          clearInterval(this.timer);
          this.codeMsg = "Send";
          this.time = 60;
          this.flag = false;
        }
      }, 1000);
    },
    forget() {
      var reg = /^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/ ;
      if (!reg.test(this.username)) {
        this.$toast({ type: "error", message: "Email format is incorrect" });
        return false;
      }
      if (this.code.trim().length != 6) {
        this.$toast({ type: "error", message: "Please enter the 6-digit verification code" });
        return false;
      }
      if (this.password.trim().length <6) {
        this.$toast({ type: "error", message: "Password cannot be less than 6 digits" });
        return false;
      }
      const user = {
        username: this.username,
        password: this.password,
        code: this.code
      };
      this.axios.put("/api/users/password", user).then(({ data }) => {
        if (data.flag) {
          this.$toast({ type: "success", message: data.message });
        } else {
          this.$toast({ type: "error", message: data.message });
        }
      });
    }
  },
  computed: {
    forgetFlag: {
      set(value) {
        this.$store.state.forgetFlag = value;
      },
      get() {
        return this.$store.state.forgetFlag;
      }
    },
    isMobile() {
      const clientWidth = document.documentElement.clientWidth;
      if (clientWidth > 960) {
        return false;
      }
      return true;
    }
  },
  watch: {
    username(value) {
      var reg = /^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/;
      if (reg.test(value)) {
        this.flag = false;
      } else {
        this.flag = true;
      }
    }
  }
};
</script>
