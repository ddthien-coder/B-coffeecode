<template>
  <div class="reply-input-wrapper" style="display: none" ref="reply">
    <textarea
      class="comment-textarea"
      :placeholder="'Reply @'+ nickname +':'"
      auto-grow
      dense
      v-model="commentContent"
    />
    <div class="emoji-container">
      <span
        :class="chooseEmoji?'emoji-btn-active':'emoji-btn'"
        @click="chooseEmoji = !chooseEmoji"
      >
        <i class="iconfont iconbiaoqing" />
      </span>
      <div style="margin-left:auto">
        <button @click="cancleReply" class="cancle-btn v-comment-btn">
          cancel
        </button>
        <button @click="insertReply" class="upload-btn v-comment-btn">
          submit
        </button>
      </div>
    </div>
    <!-- Emoticon box -->
    <emoji @addEmoji="addEmoji" :chooseEmoji="chooseEmoji"></emoji>
  </div>
</template>

<script>
import Emoji from "./Emoji";
import EmojiList from "../assets/js/emoji";
export default {
  components: {
    Emoji
  },
  data: function() {
    return {
      index: 0,
      chooseEmoji: false,
      nickname: "",
      replyId: null,
      parentId: null,
      commentContent: ""
    };
  },
  methods: {
    cancleReply() {
      this.$refs.reply.style.display = "none";
    },
    insertReply() {
      //Judgment login
      if (!this.$store.state.userId) {
        this.$store.state.loginFlag = true;
        return false;
      }
      if (this.commentContent.trim() == "") {
        this.$toast({ type: "error", message: "Reply cannot be empty" });
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
      const path = this.$route.path;
      const arr = path.split("/");
      var comment = {
        articleId: arr[2],
        replyId: this.replyId,
        parentId: this.parentId,
        commentContent: this.commentContent
      };
      this.commentContent = "";
      this.axios.post("/api/comments", comment).then(({ data }) => {
        if (data.flag) {
          this.$emit("reloadReply", this.index);
          this.$toast({ type: "success", message: "reply success" });
        } else {
          this.$toast({ type: "error", message: data.message });
        }
      });
    },
    addEmoji(text) {
      this.commentContent += text;
    }
  }
};
</script>

<style scoped>
.reply-input-wrapper {
  border: 1px solid #f0f0f0;
  border-radius: 4px;
  padding: 10px;
  margin: 0 0 10px;
}
</style>
