<template>
  <v-navigation-drawer
    app
    v-model="drawer"
    width="250"
    disable-resize-watcher
    right
    overlay-opacity="0.8"
  >
    <!-- Blogger introduction -->
    <div class="blogger-info">
      <v-avatar size="110" style="margin-bottom:0.5rem">
        <img :src="this.$store.state.blogInfo.avatar" />
      </v-avatar>
    </div>
    <!-- Blog information -->
    <div class="blog-info-wrapper">
      <div class="blog-info-data">
        <router-link to="/archives">
          <div style="font-size: 0.875rem">Article</div>
          <div style="font-size: 1.125rem;">
            {{ this.$store.state.blogInfo.articleCount }}
          </div>
        </router-link>
      </div>
      <div class="blog-info-data">
        <router-link to="/categories">
          <div style="font-size: 0.875rem">Classification</div>
          <div style="font-size: 1.125rem">
            {{ this.$store.state.blogInfo.categoryCount }}
          </div>
        </router-link>
      </div>
      <div class="blog-info-data">
        <router-link to="/tags">
          <div style="font-size: 0.875rem">Tags</div>
          <div style="font-size: 1.125rem">
            {{ this.$store.state.blogInfo.tagCount }}
          </div>
        </router-link>
      </div>
    </div>
    <hr />
    <!-- Page navigation -->
    <div class="menu-container">
      <div class="menus-item">
        <router-link to="/">
          <i class="iconfont iconzhuye" />Home
        </router-link>
      </div>
      <div class="menus-item">
        <router-link to="/archives">
          <i class="iconfont iconguidang" />Archive
        </router-link>
      </div>
      <div class="menus-item">
        <router-link to="/categories">
          <i class="iconfont iconfenlei" />Category
        </router-link>
      </div>
      <div class="menus-item">
        <router-link to="/tags">
          <i class="iconfont iconbiaoqian" />Tag
        </router-link>
      </div>
      <div class="menus-item">
        <router-link to="/links">
          <i class="iconfont iconlianjie" />Friends
        </router-link>
      </div>
      <div class="menus-item">
        <router-link to="/about">
          <i class="iconfont iconzhifeiji" />About
        </router-link>
      </div>
      <div class="menus-item">
        <router-link to="/message">
          <i class="iconfont iconpinglunzu" />Message
        </router-link>
      </div>
      <div class="menus-item" v-if="!this.$store.state.avatar">
        <a @click="openLogin"><i class="iconfont icondenglu" /> Sign in </a>
      </div>
      <div v-else>
        <div class="menus-item">
          <router-link to="/user">
            <i class="iconfont icongerenzhongxin" />Personal Center
          </router-link>
        </div>
        <div class="menus-item">
          <a @click="logout"><i class="iconfont icontuichu" />Sign out</a>
        </div>
      </div>
    </div>
  </v-navigation-drawer>
</template>

<style scoped>
.blogger-info {
  padding: 26px 30px 0;
  text-align: center;
}
.blog-info-wrapper {
  display: flex;
  align-items: center;
  padding: 12px 10px 0;
}
.blog-info-data {
  flex: 1;
  line-height: 2;
  text-align: center;
}
hr {
  border: 2px dashed #d2ebfd;
  margin: 20px 0;
}
.menu-container {
  padding: 0 10px 40px;
  animation: 0.8s ease 0s 1 normal none running sidebarItem;
}
.menus-item a {
  padding: 6px 30px;
  display: block;
  line-height: 2;
}
.menus-item i {
  margin-right: 2rem;
}
@keyframes sidebarItem {
  0% {
    transform: translateX(200px);
  }
  100% {
    transform: translateX(0);
  }
}
</style>

<script>
export default {
  computed: {
    drawer: {
      set(value) {
        this.$store.state.drawer = value;
      },
      get() {
        return this.$store.state.drawer;
      }
    },
    isLogin() {
      return this.$store.state.userId;
    }
  },
  methods: {
    openLogin() {
      this.$store.state.loginFlag = true;
    },
    logout() {
      //Jump back to the previous page if you are in the personal center
      if (this.$route.path == "/user") {
        this.$router.go(-1);
      }
      this.axios.get("/api/logout").then(({ data }) => {
        if (data.flag) {
          this.$store.commit("logout");
          this.$toast({ type: "success", message: data.message });
        } else {
          this.$toast({ type: "error", message: data.message });
        }
      });
    }
  }
};
</script>
