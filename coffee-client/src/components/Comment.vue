<template>
  <div>
    <div class="comment-title"><i class="iconfont iconpinglunzu" />Comment</div>
    <!-- Comment box -->
    <div class="comment-input-wrapper">
      <div style="display:flex">
        <v-avatar size="40">
          <img
            v-if="this.$store.state.avatar"
            :src="this.$store.state.avatar"
          />
          <img
            v-else
            src="https://gravatar.loli.net/avatar/d41d8cd98f00b204e9800998ecf8427e?d=mp&v=1.4.14"
          />
        </v-avatar>
        <div style="width:100%" class="ml-3">
          <div class="comment-input">
            <textarea
              class="comment-textarea"
              v-model="commentContent"
              placeholder="Leave something..."
              auto-grow
              dense
            />
          </div>
          <!-- Operation button -->
          <div class="emoji-container">
            <span
              :class="chooseEmoji?'emoji-btn-active':'emoji-btn'"
              @click="chooseEmoji = !chooseEmoji"
            >
              <i class="iconfont iconbiaoqing" />
            </span>
            <button
              @click="insertComment"
              class="upload-btn v-comment-btn"
              style="margin-left:auto"
            >
              submit
            </button>
          </div>
          <!-- Emoticon box -->
          <emoji @addEmoji="addEmoji" :chooseEmoji="chooseEmoji" />
        </div>
      </div>
    </div>
    <!-- Comment details -->
    <div v-if="count> 0 && reFresh">
      <!-- Number of comments -->
      <div class="count">{{ count }} comments</div>
      <!-- Comment list -->
      <div
        style="display:flex"
        class="pt-5"
        v-for="(item, index) of commentList"
        :key="item.id"
      >
        <!-- Avatar -->
        <v-avatar size="40" class="comment-avatar">
          <img :src="item.avatar" />
        </v-avatar>
        <div class="comment-meta">
          <!-- Username -->
          <div class="comment-user">
            <span v-if="!item.webSite">{{ item.nickname }}</span>
            <a v-else :href="item.webSite" target="_blank">
              {{ item.nickname }}
            </a>
            <span class="blogger-tag" v-if="item.userId == 1">Blogger</span>
          </div>
          <!-- Information -->
          <div class="comment-info">
            <!-- Floor -->
            <span style="margin-right:10px">{{ count-index }} floor</span>
            <!-- Publication time -->
            <span style="margin-right:10px">{{ item.createTime | date }}</span>
            <!-- Like -->
            <span
              :class="isLike(item.id) + 'iconfont icondianzan'"
              @click="like(item)"
            />
            <span v-show="item.likeCount> 0"> {{ item.likeCount }}</span>
            <!-- Reply -->
            <span class="reply-btn" @click="replyComment(index, item)">
              Reply
            </span>
          </div>
        <!-- Comment content -->
          <p v-html="item.commentContent" class="comment-content"></p>
          <!-- Reply person -->
          <div
            style="display:flex"
            v-for="reply of item.replyDTOList"
            :key="reply.id"
          >
            <!-- Avatar -->
            <v-avatar size="36" class="comment-avatar">
              <img :src="reply.avatar" />
            </v-avatar>
            <div class="reply-meta">
              <!-- Username -->
              <div class="comment-user">
                <span v-if="!reply.webSite">{{ reply.nickname }}</span>
                <a v-else :href="reply.webSite" target="_blank">
                  {{ reply.nickname }}
                </a>
                <span class="blogger-tag" v-if="reply.userId == 1">Blogger</span>
              </div>
              <!-- Information -->
              <div class="comment-info">
                <!-- Publication time -->
                <span style="margin-right:10px">
                  {{ reply.createTime | date }}
                </span>
                <!-- Like -->
                <span
                  :class="isLike(reply.id) + 'iconfont icondianzan'"
                  @click="like(reply)"
                />
                <span v-show="reply.likeCount> 0"> {{ reply.likeCount }}</span>
                <!-- Reply -->
                <span class="reply-btn" @click="replyComment(index, reply)">
                  Reply
                </span>
              </div>
              <!-- Reply content -->
              <p class="comment-content">
                <!-- Reply user name -->
                <template v-if="reply.replyId != item.userId">
                  <span v-if="!reply.replyWebSite" class="ml-1">
                    @{{ reply.replyNickname }}
                  </span>
                  <a
                    v-else
                    :href="reply.replyWebSite"
                    target="_blank"
                    class="comment-nickname ml-1"
                  >
                    @{{ reply.replyNickname }}
                  </a>
                  ,
                </template>
                <span v-html="reply.commentContent" />
              </p>
            </div>
          </div>
          <!-- Number of responses -->
          <div
            class="mb-3"
            style="font-size:0.75rem;color:#6d757a"
            v-show="item.replyCount> 3"
            ref="check"
          >
            Total
            <b>{{ item.replyCount }}</b>
            Replies,
            <span
              style="color:#00a1d6;cursor:pointer"
              @click="checkReplies(index, item)"
            >
              Click to view
            </span>
          </div>
          <!-- Reply to pagination -->
          <div
            class="mb-3"
            style="font-size:0.75rem;color:#222;display:none"
            ref="paging"
          >
            <span style="padding-right:10px">
              A total of {{ Math.ceil(item.replyCount / 5) }} pages
            </span>
            <paging
              ref="page"
              :totalPage="Math.ceil(item.replyCount / 5)"
              :index="index"
              :commentId="item.id"
              @changeReplyCurrent="changeReplyCurrent"
            />
          </div>
          <!-- Reply box -->
          <Reply ref="reply" @reloadReply="reloadReply" />
        </div>
      </div>
      <!-- Load button -->
      <div class="load-wrapper">
        <v-btn outlined v-if="count> commentList.length" @click="listComments">
          load more...
        </v-btn>
      </div>
    </div>
    <!-- No comment prompt -->
    <div v-else style="padding:1.25rem;text-align:center">
      Come and post a comment~
    </div>
  </div>
</template>

<script>
import Reply from "./Reply";
import Paging from "./Paging";
import Emoji from "./Emoji";
import EmojiList from "../assets/js/emoji";
export default {
  components: {
    Reply,
    Emoji,
    Paging
  },
  props: {
    commentList: {
      type: Array
    },
    count: {
      type: Number
    }
  },
  data: function() {
    return {
      reFresh: true,
      commentContent: "",
      chooseEmoji: false,
      current: 1
    };
  },
  methods: {
    replyComment(index, item) {
      this.$refs.reply.forEach(item => {
        item.$el.style.display = "none";
      });
      //Pass value to reply box
      this.$refs.reply[index].commentContent = "";
      this.$refs.reply[index].nickname = item.nickname;
      this.$refs.reply[index].replyId = item.userId;
      this.$refs.reply[index].parentId = this.commentList[index].id;
      this.$refs.reply[index].chooseEmoji = false;
      this.$refs.reply[index].index = index;
      this.$refs.reply[index].$el.style.display = "block";
    },
    addEmoji(key) {
      this.commentContent += key;
    },
    checkReplies(index, item) {
      this.axios
        .get("/api/comments/replies", {
          params: { current: 1, commentId: item.id }
        })
        .then(({ data }) => {
          this.$refs.check[index].style.display = "none";
          item.replyDTOList = data.data;
          //Show pagination when more than 1 page
          if (Math.ceil(item.replyCount / 5) > 1) {
            this.$refs.paging[index].style.display = "flex";
          }
        });
    },
    changeReplyCurrent(current, index, commentId) {
      //View the next page of responses
      this.axios
        .get("/api/comments/replies", {
          params: { current: current, commentId: commentId }
        })
        .then(({ data }) => {
          this.commentList[index].replyDTOList = data.data;
        });
    },
    listComments() {
      //View next page of comments
      this.current++;
      const path = this.$route.path;
      const arr = path.split("/");
      this.axios
        .get("/api/comments", {
          params: { current: this.current, articleId: arr[2] }
        })
        .then(({ data }) => {
          this.commentList.push(...data.data.recordList);
        });
    },
    insertComment() {
//Judgment login
      if (!this.$store.state.userId) {
        this.$store.state.loginFlag = true;
        return false;
      }
      //Dismissal
      if (this.commentContent.trim() == "") {
        this.$toast({ type: "error", message: "Comments cannot be empty" });
        return false;
      }
      //Analyze emoji
      var reg = /\[.+?\]/g;
      this.commentContent = this.commentContent.replace(reg, function(str) {
        return (
          "<img src='" +
          EmojiList[str] +
          "'width='22'height='20' style='padding: 0 1px'/>"
        );
      });
      //send request
      const path = this.$route.path;
      const arr = path.split("/");
      var comment = {
        articleId: arr[2],
        commentContent: this.commentContent
      };
      this.commentContent = "";
      this.axios.post("/api/comments", comment).then(({ data }) => {
        if (data.flag) {
          //Check the latest comments
          this.$emit("reloadComment");
          this.$toast({ type: "success", message: data.message });
        } else {
          this.$toast({ type: "error", message: data.message });
        }
      });
    },
    like(comment) {
      //Judgment login
      if (!this.$store.state.userId) {
        this.$store.state.loginFlag = true;
        return false;
      }
      //send request
      let param = new URLSearchParams();
      param.append("commentId", comment.id);
      this.axios.post("/api/comments/like", param).then(({ data }) => {
        if (data.flag) {
          //Determine whether to like
          if (this.$store.state.commentLikeSet.indexOf(comment.id) != -1) {
            this.$set(comment, "likeCount", comment.likeCount-1);
          } else {
            this.$set(comment, "likeCount", comment.likeCount + 1);
          }
          this.$store.commit("commentLike", comment.id);
        }
      });
    },
    reloadReply(index) {
      this.axios
        .get("/api/comments/replies", {
          params: {
            current: this.$refs.page[index].current,
            commentId: this.commentList[index].id
          }
        })
        .then(({ data }) => {
          this.commentList[index].replyCount++;
          //Reply more than 5 display paging
          if (this.commentList[index].replyCount > 5) {
            this.$refs.paging[index].style.display = "flex";
          }
          this.$refs.check[index].style.display = "none";
          this.$refs.reply[index].$el.style.display = "none";
          this.commentList[index].replyDTOList = data.data;
        });
    }
  },
  computed: {
    isLike() {
      return function(commentId) {
        var commentLikeSet = this.$store.state.commentLikeSet;
        return commentLikeSet.indexOf(commentId) != -1 ? "like-active" : "like";
      };
    }
  },
  watch: {
    commentList() {
      this.reFresh = false;
      this.$nextTick(() => {
        this.reFresh = true;
      });
    }
  }
};
</script>

<style scoped>
p {
  margin-bottom: 1.25rem !important;
}
.blogger-tag {
  background: #ffa51e;
  font-size: 12px;
  display: inline-block;
  border-radius: 2px;
  color: #fff;
  padding: 0 5px;
  margin-left: 6px;
}
.comment-title {
  display: flex;
  align-items: center;
  font-size: 1.25rem;
  font-weight: bold;
  line-height: 40px;
  margin-bottom: 10px;
}
.comment-title i {
  font-size: 1.5rem;
  margin-right: 5px;
}
.comment-input-wrapper {
  border: 1px solid #f0f0f0;
  border-radius: 4px;
  padding: 10px;
  margin: 0 0 10px;
}
.count {
  padding: 5px;
  line-height: 1.75;
  font-size: 1.25rem;
  font-weight: bold;
}
.comment-meta {
  margin-left: 0.8rem;
  width: 100%;
  border-bottom: 1px dashed #f5f5f5;
}
.reply-meta {
  margin-left: 0.8rem;
  width: 100%;
}
.comment-user {
  font-size: 14px;
  line-height: 1.75;
}
.comment-user a {
  color: #1abc9c !important;
  font-weight: 500;
  transition: 0.3s all;
}
.comment-nickname {
  text-decoration: none;
  color: #1abc9c !important;
  font-weight: 500;
}
.comment-info {
  line-height: 1.75;
  font-size: 0.75rem;
  color: #b3b3b3;
}
.reply-btn {
  cursor: pointer;
  float: right;
  color: #ef2f11;
}
.comment-content {
  display: flex;
  font-size: 0.875rem;
  line-height: 1.75;
  padding-top: 0.625rem;
}
.comment-avatar {
  transition: all 0.5s;
}
.comment-avatar:hover {
  transform: rotate(360deg);
}
.like {
  cursor: pointer;
  font-size: 0.875rem;
}
.like:hover {
  color: #eb5055;
}
.like-active {
  cursor: pointer;
  font-size: 0.875rem;
  color: #eb5055;
}
.load-wrapper {
  margin-top: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.load-wrapper button {
  background-color: #49b1f5;
  color: #fff;
}
</style>
