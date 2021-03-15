<template>
  <!-- Search box -->
  <v-dialog v-model="searchFlag" max-width="600" :fullscreen="isMobile">
    <v-card class="search-wrapper" style="border-radius:4px">
      <div class="mb-3">
        <span class="search-title">Local Search</span>
        <!-- Close button -->
        <v-icon class="float-right" @click="searchFlag = false">
          mdi-close
        </v-icon>
      </div>
      <!-- Input box -->
      <div class="search-input-wrapper">
        <v-icon>mdi-magnify</v-icon>
        <input v-model="keywords" placeholder="Enter the title or content of the article..." />
      </div>
      <!-- search result -->
      <div class="search-result-wrapper">
        <hr class="divider" />
        <ul>
          <li class="search-reslut" v-for="item of articleList" :key="item.id">
            <!-- Article title -->
            <a @click="goTo(item.id)" v-html="item.articleTitle" />
            <!-- Article content -->
            <p
              class="search-reslut-content text-justify"
              v-html="item.articleContent"
            />
          </li>
        </ul>
        <!-- There is no prompt in the search result -->
        <div
          v-show="flag && articleList.length == 0"
          style="font-size:0.875rem"
        >
          Could not find the content you are looking for: {{ keywords }}
        </div>
      </div>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  data: function() {
    return {
      keywords: "",
      articleList: [],
      flag: false
    };
  },
  methods: {
    goTo(articleId) {
      this.$store.state.searchFlag = false;
      this.$router.push({ path: "/articles/" + articleId });
    }
  },
  computed: {
    searchFlag: {
      set(value) {
        this.$store.state.searchFlag = value;
      },
      get() {
        return this.$store.state.searchFlag;
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
    keywords(value) {
      this.flag = value.trim() != "" ? true : false;
      this.axios
        .get("/api/articles/search", {
          params: { current: 1, keywords: value }
        })
        .then(({ data }) => {
          this.articleList = data.data;
        });
    }
  }
};
</script>

<style scoped>
.search-wrapper {
  padding: 1.25rem;
  height: 100%;
}
.search-title {
  color: #49b1f5;
  font-size: 1.25rem;
  line-height: 1;
}
.search-input-wrapper {
  display: flex;
  padding: 5px;
  height: 35px;
  width: 100%;
  border: 2px solid #8e8cd8;
  border-radius: 2rem;
}
.search-input-wrapper input {
  width: 100%;
  margin-left: 5px;
  outline: none;
}
@media (min-width: 960px) {
  .search-result-wrapper {
    padding-right: 5px;
    height: 450px;
    overflow: auto;
  }
}
@media (max-width: 959px) {
  .search-result-wrapper {
    height: calc(100vh - 110px);
    overflow: auto;
  }
}
.search-reslut a {
  color: #555;
  font-weight: bold;
  border-bottom: 1px solid #999;
  text-decoration: none;
}
.search-reslut-content {
  color: #555;
  cursor: pointer;
  border-bottom: 1px dashed #ccc;
  padding: 5px 0;
  line-height: 2;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}
.divider {
  margin: 20px 0;
  border: 2px dashed #d2ebfd;
}
</style>
