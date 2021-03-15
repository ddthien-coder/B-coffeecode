<template>
  <div>
    <!-- Cover image -->
    <div class="banner" :style="articleCover">
      <div class="article-info-container">
        <!-- Article title -->
        <div class="article-title">{{ article.articleTitle }}</div>
        <div class="article-info">
          <div class="first-line">
            <!-- Publication time -->
            <span>
              <i class="iconfont iconrili" />
              Posted on {{ article.createTime | date }}
            </span>
            <span class="separator">|</span>
            <!-- Publication time -->
            <span>
              <i class="iconfont icongengxinshijian" />
              Updated on
              <template v-if="article.updateTime">
                {{ article.updateTime | date }}
              </template>
              <template v-else>
                {{ article.createTime | date }}
              </template>
            </span>
            <span class="separator">|</span>
            <!-- Article classification -->
            <span class="article-category">
              <i class="iconfont iconfenlei1" />
              <router-link :to="'/categories/' + article.categoryId">
                {{ article.categoryName }}
              </router-link>
            </span>
          </div>
          <div class="second-line">
            <!-- Word count -->
            <span>
              <i class="iconfont iconzishu" />
              Word count: {{ wordNum | num }}
            </span>
            <span class="separator">|</span>
            <!-- Reading time -->
            <span>
              <i class="iconfont iconshijian" />
              Reading time: {{ readTime }}
            </span>
          </div>
          <div class="third-line">
            <span class="separator">|</span>
            <!-- Reading volume -->
            <span>
              <i class="iconfont iconliulan" /> Reading volume: {{ article.viewsCount }}
            </span>
            <span class="separator">|</span>
            <!-- Number of comments -->
            <span>
              <i class="iconfont iconpinglunzu1" />Number of comments:
              <template v-if="count">{{ count }}</template>
              <template v-else>0</template>
            </span>
          </div>
        </div>
      </div>
    </div>
    <!-- Content -->
    <v-card class="blog-container">
      <article
        class="article-content markdown-body"
        v-html="article.articleContent"
        ref="article"
      />
      <!-- Copyright statement -->
      <div class="aritcle-copyright">
        <div>
          <Span> article Author: </span>
          <a href="https://www.coffeecode.com" target="_blank"> Coffee Code</a>
        </div>
        <div>
          <span>Article link:</span>
          <a :href="articleHref" target="_blank">{{ articleHref }} </a>
        </div>
      </div>
      <!-- Forward -->
      <div class="article-operation">
        <div class="tag-container">
          <router-link
            v-for="item of article.tagDTOList"
            :key="item.id"
            :to="'/tags/' + item.id"
          >
            {{ item.tagName }}
          </router-link>
        </div>
        <share style="margin-left:auto" :config="config" />
      </div>
      <!-- Like, reward, etc. -->
      <div class="article-reward">
        <!-- Like button -->
        <a :class="isLike" @click="like">
          <v-icon size="14" color="#fff">mdi-thumb-up</v-icon> Like
          <span v-show="article.likeCount> 0">{{ article.likeCount }}</span>
        </a>
        <a class="reward-btn">
          <!-- Reward button -->
          <i class="iconfont iconerweima" /> Reward
        </a>
      </div>
      <!-- Comment -->
      <comment
        :commentList="commentList"
        :count="count"
        @reloadComment="listComment"
      />
    </v-card>
  </div>
</template>

<script>
import Clipboard from "clipboard";
import Comment from "../../components/Comment";
export default {
  components: {
    Comment
  },
  created() {
    this.getArticle();
    this.listComment();
  },
  destroyed() {
    this.clipboard.destroy();
  },
  data: function() {
    return {
      config: {
        sites: ["qzone", "wechat", "weibo", "qq"]
      },
      imgList: [],
      article: {},
      commentList: [],
      count: 0,
      wordNum: "",
      readTime: "",
      articleHref: window.location.href
    };
  },
  methods: {
    getArticle() {
      const that = this;
//Query article
      this.axios.get("/api" + this.$route.path).then(({ data }) => {
        //Convert markdown to Html
        this.markdownToHtml(data.data);
        this.$nextTick(() => {
          //Count the number of words in the article
          this.wordNum = this.deleteHTMLTag(this.article.articleContent).length;
          //Calculate reading time
          this.readTime = Math.round(this.wordNum / 400) + "minute";
          //Add code copy function
          this.clipboard = new Clipboard(".copy-btn");
          this.clipboard.on("success", () => {
            this.$toast({ type: "success", message: "Copy successful" });
          });
          //Add picture preview function
          const imgList = this.$refs.article.getElementsByTagName("img");
          for (var i = 0; i <imgList.length; i++) {
            this.imgList.push(imgList[i].src);
            imgList[i].style.cssText = "cursor:zoom-in;";
            imgList[i].addEventListener("click", function(e) {
              that.previewImg(e.toElement.src);
            });
          }
        });
      });
    },
    listComment() {
      const path = this.$route.path;
      const arr = path.split("/");
      const articleId = arr[arr.length-1];
      this.axios
        .get("/api/comments", {
          params: {current: 1, articleId: articleId}
        })
        .then(({ data }) => {
          this.commentList = data.data.recordList;
          this.count = data.data.count;
        });
    },
    like() {
      //Judgment login
      if (!this.$store.state.userId) {
        this.$store.state.loginFlag = true;
        return false;
      }
      //send request
      let param = new URLSearchParams();
      param.append("articleId", this.article.id);
      this.axios.post("/api/articles/like", param).then(({ data }) => {
        if (data.flag) {
          //Determine whether to like
          if (this.$store.state.articleLikeSet.indexOf(this.article.id) != -1) {
            this.$set(this.article, "likeCount", this.article.likeCount-1);
          } else {
            this.$set(this.article, "likeCount", this.article.likeCount + 1);
          }
          this.$store.commit("articleLike", this.article.id);
        }
      });
    },
    markdownToHtml(article) {
      const MarkdownIt = require("markdown-it");
      const hljs = require("highlight.js");
      const md = new MarkdownIt({
        html: true,
        linkify: true,
        typographer: true,
        highlight: function(str, lang) {
          // The current time plus a random number to generate a unique id identifier
          const codeIndex = parseInt(Date.now());
          // The copy function mainly uses clipboard.js
          let html = `<button class="copy-btn iconfont iconfuzhi" type="button" data-clipboard-action="copy" data-clipboard-target="#copy${codeIndex}"></button>`;
          const linesLength = str.split(/\n/).length-1;
          // Generate line number
          let linesNum ='<span aria-hidden="true" class="line-numbers-rows">';
          for (let index = 0; index <linesLength; index++) {
            linesNum = linesNum + "<span></span>";
          }
          linesNum += "</span>";
          if (lang && hljs.getLanguage(lang)) {
            // highlight.js highlight code
            const preCode = hljs.highlight(lang, str, true).value;
            html = html + preCode;
            if (linesLength) {
              html +='<b class="name">' + lang + "</b>";
            }
            // Wrap the code in textarea. To prevent textarea rendering problems, replace "<" with "<" here, which does not affect the copy function
            return `<pre class="hljs"><code>${html}</code>${linesNum}</pre><textarea style="position: absolute;top: -9999px;left: -9999px;z- index: -9999;" id="copy${codeIndex}">${str.replace(
              /<\/textarea>/g,
              "</textarea>"
            )}</textarea>`;
          }
        }
      });
      //Replace markdown with html tags
      article.articleContent = md.render(article.articleContent);
      this.article = article;
    },
    previewImg(img) {
      this.$imagePreview({
        images: this.imgList,
        index: this.imgList.indexOf(img)
      });
    },
    deleteHTMLTag(content) {
      return content
        .replace(/<\/?[^>]*>/g, "")
        .replace(/[|]*\n/, "")
        .replace(/&npsp;/gi, "");
    }
  },
  computed: {
    articleCover() {
      return (
        "background: url(" +
        this.article.articleCover +
        ") center center / cover no-repeat"
      );
    },
    isLike() {
      var articleLikeSet = this.$store.state.articleLikeSet;
      return articleLikeSet.indexOf(this.article.id) != -1
        ? "like-btn-active"
        : "like-btn";
    }
  }
};
</script>

<style scoped>
.banner:before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}
.article-info i {
  font-size: 14px;
}
.article-info {
  font-size: 14px;
  line-height: 1.75;
  display: inline-block;
}
.article-container:hover {
  box-shadow: 0 4px 12px 12px rgba(7, 17, 27, 0.15);
}
@media (min-width: 760px) {
  .banner {
    color: #eee !important;
  }
  .article-info span {
    font-size: 95%;
  }
  .article-info-container {
    position: absolute;
    bottom: 1.9rem;
    padding: 0 8%;
    width: 100%;
  }
  .second-line,
  .third-line {
    display: inline;
  }
  .article-title {
    font-size: 1.875rem;
    margin-bottom: 0.4rem;
  }
}
@media (max-width: 759px) {
  .banner {
    color: #eee !important;
    height: 360px;
  }
  .article-info span {
    font-size: 90%;
  }
  .separator:first-child {
    display: none;
  }
  .blog-container {
    margin: 322px 5px 0 5px;
  }
  .article-info-container {
    position: absolute;
    bottom: 1.3rem;
    padding: 0 5%;
    width: 100%;
    color: #eee;
    text-align: left;
  }
  .article-title {
    font-size: 1.5rem;
    margin-bottom: 0.4rem;
  }
}
.article-content {
  word-break: break-word;
  font-size: 1rem;
  line-height: 1.8;
}
.article-operation {
  display: flex;
  align-items: center;
}
.article-category a {
  color: #fff !important;
}
.tag-container a {
  display: inline-block;
  margin: 0.5rem 0.5rem 0.5rem 0;
  padding: 0 0.75rem;
  width: fit-content;
  border: 1px solid #49b1f5;
  border-radius: 1rem;
  color: #49b1f5 !important;
  font-size: 12px;
  line-height: 2;
}
.tag-container a:hover {
  color: #fff !important;
  background: #49b1f5;
  transition: all 0.5s;
}
.aritcle-copyright {
  position: relative;
  margin-top: 40px;
  margin-bottom: 10px;
  font-size: 0.875rem;
  line-height: 2;
  padding: 0.625rem 1rem;
  border: 1px solid #eee;
}
.aritcle-copyright span {
  color: #49b1f5;
  font-weight: bold;
}
.aritcle-copyright a {
  text-decoration: underline !important;
  color: #99a9bf !important;
}
.aritcle-copyright:before {
  position: absolute;
  top: 0.7rem;
  right: 0.7rem;
  width: 1rem;
  height: 1rem;
  border-radius: 1rem;
  background: #49b1f5;
  content: "";
}
.aritcle-copyright:after {
  position: absolute;
  top: 0.95rem;
  right: 0.95rem;
  width: 0.5rem;
  height: 0.5rem;
  border-radius: 0.5em;
  background: #fff;
  content: "";
}
.article-reward {
  margin: 5rem 0;
  display: flex;
  justify-content: center;
  align-items: center;
}
.reward-btn {
  position: relative;
  display: inline-block;
  width: 100px;
  background: #49b1f5;
  margin: 0 1rem;
  color: #fff !important;
  text-align: center;
  line-height: 36px;
  font-size: 0.875rem;
}
.reward-btn:hover .reward-main {
  display: block;
}
.reward-main {
  display: none;
  position: absolute;
  bottom: 40px;
  left: 0;
  margin: 0;
  padding: 0 0 15px;
  width: 100%;
}
.reward-all {
  display: inline-block;
  margin: 0 0 0 -110px;
  padding: 20px 10px 8px !important;
  width: 320px;
  border-radius: 4px;
  background: #f5f5f5;
}
.reward-all:before {
  position: absolute;
  bottom: -10px;
  left: 0;
  width: 100%;
  height: 20px;
  content: "";
}
.reward-all:after {
  content: "";
  position: absolute;
  right: 0;
  bottom: 2px;
  left: 0;
  margin: 0 auto;
  width: 0;
  height: 0;
  border-top: 13px solid #f5f5f5;
  border-right: 13px solid transparent;
  border-left: 13px solid transparent;
}
.reward-item {
  display: inline-block;
  padding: 0 8px;
  list-style-type: none;
}
.reward-img {
  width: 130px;
  height: 130px;
  display: block;
}
.reward-desc {
  margin: -5px 0;
  color: #858585;
  text-align: center;
}
.like-btn {
  display: inline-block;
  width: 100px;
  background: #969696;
  color: #fff !important;
  text-align: center;
  line-height: 36px;
  font-size: 0.875rem;
}
.like-btn-active {
  display: inline-block;
  width: 100px;
  background: #ec7259;
  color: #fff !important;
  text-align: center;
  line-height: 36px;
  font-size: 0.875rem;
}
</style>

<style lang="scss">
pre.hljs {
  padding: 12px 2px 12px 40px !important;
  border-radius: 5px !important;
  position: relative;
  font-size: 14px !important;
  line-height: 22px !important;
  overflow: hidden !important;
  &:hover .copy-btn {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  code {
    display: block !important;
    margin: 0 10px !important;
    overflow-x: auto !important;
    &::-webkit-scrollbar {
      z-index: 11;
      width: 6px;
    }
    &::-webkit-scrollbar:horizontal {
      height: 6px;
    }
    &::-webkit-scrollbar-thumb {
      border-radius: 5px;
      width: 6px;
      background: #666;
    }
    &::-webkit-scrollbar-corner,
    &::-webkit-scrollbar-track {
      background: #1e1e1e;
    }
    &::-webkit-scrollbar-track-piece {
      background: #1e1e1e;
      width: 6px;
    }
  }
  .line-numbers-rows {
    position: absolute;
    pointer-events: none;
    top: 12px;
    bottom: 12px;
    left: 0;
    font-size: 100%;
    width: 40px;
    text-align: center;
    letter-spacing: -1px;
    border-right: 1px solid rgba(0, 0, 0, 0.66);
    user-select: none;
    counter-reset: linenumber;
    span {
      pointer-events: none;
      display: block;
      counter-increment: linenumber;
      &:before {
        content: counter(linenumber);
        color: #999;
        display: block;
        text-align: center;
      }
    }
  }
  b.name {
    position: absolute;
    top: 7px;
    right: 45px;
    z-index: 1;
    color: #999;
    pointer-events: none;
  }
  .copy-btn {
    position: absolute;
    top: 6px;
    right: 6px;
    z-index: 1;
    color: #ccc;
    background-color: #525252;
    border-radius: 6px;
    display: none;
    font-size: 14px;
    width: 32px;
    height: 24px;
    outline: none;
  }
}
</style>
