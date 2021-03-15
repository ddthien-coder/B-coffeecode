<template>
  <div class="rightside" :style="isShow">
    <div :class="'rightside-config-hide ' + isOut">
      <i :class="'iconfont rightside-icon ' + icon" @click="check" />
    </div>
    <div class="setting-container" @click="show">
      <i class="iconfont iconshezhi setting" />
    </div>
    <i @click="backTop" class="iconfont rightside-icon iconziyuanldpi" />
  </div>
</template>

<script>
export default {
  mounted() {
    window.addEventListener("scroll", this.scrollToTop);
  },
  destroyed() {
    window.removeEventListener("scroll", this.scrollToTop);
  },
  data: function() {
    return {
      isShow: "",
      isOut: "rightside-out",
      icon: "iconyueliang"
    };
  },
  methods: {
    // Back to the top method
    backTop() {
      window.scrollTo({
        behavior: "smooth",
        top: 0
      });
    },
    // In order to calculate the height from the top, when the height is greater than 100, the top icon will be displayed, and the icon will be hidden if it is less than 100
    scrollToTop() {
      const that = this;
      let scrollTop =
        window.pageYOffset ||
        document.documentElement.scrollTop ||
        document.body.scrollTop;
      that.scrollTop = scrollTop;
      if (that.scrollTop > 20) {
        that.isShow = "opacity: 1;transform: translateX(-38px);";
      } else {
        that.isShow = "";
      }
    },
    show() {
      const flag = this.isOut == "rightside-out";
      this.isOut = flag ? "rightside-in" : "rightside-out";
    },
    check() {
      const flag = this.icon == "iconyueliang";
      this.icon = flag ? "icontaiyang" : "iconyueliang";
      this.$vuetify.theme.dark = !this.$vuetify.theme.dark;
    }
  }
};
</script>

<style scoped>
.rightside {
  z-index: 4;
  position: fixed;
  right: -38px;
  bottom: 40px;
  transition: all 0.5s;
}
.rightside-config-hide {
  transform: translate(35px, 0);
}
.rightside-out {
  animation: rightsideOut 0.3s;
}
.rightside-in {
  transform: translate(0, 0) !important;
  animation: rightsideIn 0.3s;
}
.rightside-icon,
.setting-container {
  display: block;
  margin-bottom: 2px;
  width: 30px;
  height: 30px;
  background-color: #49b1f5;
  color: #fff;
  text-align: center;
  font-size: 16px;
  line-height: 30px;
  cursor: pointer;
}
.rightside-icon:hover,
.setting-container:hover {
  background-color: #ff7242;
}
.setting-container i {
  display: block;
  animation: turn-around 2s linear infinite;
}
@keyframes turn-around {
  0% {
    transform: rotate(0);
  }
  100% {
    transform: rotate(360deg);
  }
}
@keyframes rightsideOut {
  0% {
    transform: translate(0, 0);
  }
  100% {
    transform: translate(30px, 0);
  }
}
@keyframes rightsideIn {
  0% {
    transform: translate(30px, 0);
  }
  100% {
    transform: translate(0, 0);
  }
}
</style>
