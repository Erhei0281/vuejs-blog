<template>
  <!-- Check blog posts exist -->
  <div v-if="posts.length !== 0" class="blog-main">
    <!-- Template for blog posts -->
    <div v-for="post in posts" :key="post.id" v-bind:post="post" class="blog-post">
      <router-link :to="linkResolver(post)">
        <h2>{{ $prismic.richTextAsPlain(post.data.title) }}</h2>
        <p class="blog-post-meta">
          <span
            class="created-at"
          >{{ Intl.DateTimeFormat('en-US', dateOptions).format(new Date(post.data.release_date)) }}</span>
        </p>
        <div>
          <p>{{getFirstParagraph(post)}}</p>
        </div>
      </router-link>
    </div>
  </div>
  <!-- If no blog posts return message -->
  <div v-else class="blog-main">
    <p>No Posts published at this time.</p>
  </div>
</template>

<script>
export default {
  name: "blog-posts",
  data() {
    return {
      posts: [],
      dateOptions: { year: "numeric", month: "short", day: "2-digit" },
      linkResolver: this.$prismic.linkResolver
    };
  },
  methods: {
    async getPosts() {
      //Query to get blog posts
      await this.$prismic.client
        .query(this.$prismic.Predicates.at("document.type", "blog-po"), {
          orderings: "[my.post.date desc]"
        })
        .then(response => {
          this.posts = response.results;
        });
    },
    //Function to get the first paragraph of text in a blog post and limit the displayed text at 300 characters
    getFirstParagraph(post) {
      const textLimit = 300;
      const slices = post.data.body;
      let firstParagraph = "";
      let haveFirstParagraph = false;

      slices.map(function(slice) {
        if (!haveFirstParagraph) {
          if (slice.slice_type == "text") {
            slice.primary.text.forEach(function(block) {
              if (block.type == "paragraph") {
                if (!haveFirstParagraph) {
                  firstParagraph += block.text;
                  haveFirstParagraph = true;
                }
              }
            });
          }
        }
      });

      const limitedText = firstParagraph.substr(0, textLimit);

      if (firstParagraph.length > textLimit) {
        return limitedText.substr(0, limitedText.lastIndexOf(" ")) + "...";
      } else {
        return firstParagraph;
      }
    }
  },
  created() {
    this.getPosts();
  }
};
</script>

<style scoped>
.blog-main {
  max-width: 700px;
  margin: auto;
}
.blog-post {
  margin-bottom: 3rem;
}
.blog-post h2 {
  margin: 0;
}
.blog-post-meta {
  color: #9a9a9a;
  font-family: "Lato", sans-serif;
  margin-bottom: 8px;
  font-size: 16px;
}
/* Media Queries */
@media (max-width: 767px) {
  .blog-main {
    padding: 0 20px;
    font-size: 18px;
  }
}
</style>