<template>
  <v-layout row>
    <v-flex xs12>
  <v-list v-for="comment in comments" :key="comment.id">
    <v-list-item>
      {{ comment.text }}
    </v-list-item>
    <v-list-item
      >Posted by : {{ comment.User.firstName }} {{ comment.User.lastName }} at
      {{ $moment(comment.createdAt).format("DD-MM-YYYY HH:mm:SS") }}
    </v-list-item>
    <v-chip
      v-if="userId == comment.userId || isAdmin"
      class="mx-3 my-2"
      @click="deleteComment(comment.id)"
      color="error"
      depressed
      >Delete Comment</v-chip
    >
    <v-divider></v-divider>
  </v-list>
  </v-flex>
  <v-divider></v-divider>
    <v-dialog v-model="dialog" persistent max-width="290">
      <template v-slot:activator="{ on, attrs }">
        <v-flex xs12 class="ma-2">
        <v-btn class="mx-3 my-3"
        color="primary" dark v-bind="attrs" v-on="on">
          New Comment
        </v-btn>
        </v-flex>
      </template>
      <h4>New Comment</h4>
      <v-divider></v-divider>
      <v-textarea
        v-model="newComment"
        solo
        name="comment"
        label="Your Comment ..."
      ></v-textarea>
      <v-layout>
        <v-flex xs12 sm3>
        <v-spacer></v-spacer>

        <v-btn color="green darken-1" text @click="dialog = false">
          Cancel
        </v-btn></v-flex>
        <v-flex xs12 sm3>
        <v-btn color="green darken-1" text @click="addComment">
          Post Comment
        </v-btn></v-flex>
      </v-layout>
    </v-dialog>
    </v-layout>
</template>

<script>
import axios from "axios";

export default {
  name: "CommentDisplay",
  props: {
    postId: {
      type: Number,
    },
  },
  data() {
    return {
      newComment: "",
      comments: [],
      dialog: false,
    };
  },
  created() {
    let token = this.$store.state.user.token;
    axios
      .get(`http://localhost:3000/api/auth/comment/${this.postId}`, {
        headers: {
          Authorization: "Bearer " + token,
        },
      })
      .then((response) => (this.comments = response.data));
  },
  computed: {
    userId: function() {
      return this.$store.state.user.userId;
    },
    isAdmin: function() {
      return this.$store.state.user.isAdmin;
    },
  },
  methods: {
    addComment: function() {
      let token = this.$store.state.user.token;
      let userId = this.$store.state.user.userId;
      let comment = {
        postId: this.postId,
        text: this.newComment,
        user: userId,
      };
      axios({
        url: "http://localhost:3000/api/auth/comment",
        method: "POST",
        data: comment,
        headers: {
          Authorization: "Bearer " + token,
        },
      })
        .then(() => {
          window.location.reload();
        })
        .catch((err) => console.log(err));
    },
    deleteComment: function(commentId) {
      let token = localStorage.token;
      axios({
        url: `http://localhost:3000/api/auth/comment/delete/${commentId}`,
        method: "DELETE",
        headers: { Authorization: "Bearer " + token },
      })
        .then(() => {
          window.location.reload();
        })
        .catch((err) => console.log(err));
    },
  },
};
</script>
<style>
.v-dialog.v-dialog--active.v-dialog--persistent {
    background: white;
}
</style>