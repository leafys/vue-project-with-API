<template>
  <div>
    <h1>Страница с аниме постами</h1>
    <my-input
      v-focus
      :model-value="searchQuery"
      @update:model-value="setSearchQuery"
      placeholder="Поиск..."
    />
    <div class="app__btns">
      <my-button @click="showDialog">Создать аниме пост</my-button>
      <my-select
        :options="sortOptions"
        :model-value="selectedSort"
        @update:model-value="setSelectedSort"
      />
    </div>
    <my-dialog v-model:show="dialogVisible">
      <post-form @create="createPost" />
    </my-dialog>
    <post-list
      :posts="sortedAndSearchPosts"
      @remove="removePost"
      v-if="!isPostLoading"
    />
    <div v-else>идет загрузка...</div>

    <div v-intersection="loadMorePosts" class="observer"></div>
  </div>
</template>

<script>
import PostForm from '@/components/PostForm';
import PostList from '@/components/PostList';
import { mapState, mapGetters, mapActions, mapMutations } from 'vuex';

export default {
  components: {
    PostForm,
    PostList,
  },

  data() {
    return {
      dialogVisible: false,
    };
  },

  methods: {
    ...mapMutations({
      setPage: 'post/setPage',
      setSearchQuery: 'post/setSearchQuery',
      setSelectedSort: 'post/setSelectedSort',
    }),
    ...mapActions({
      loadMorePosts: 'post/loadMorePosts',
      fetchPosts: 'post/fetchPosts',
    }),

    createPost(post) {
      this.posts.push(post);
      this.dialogVisible = false;
    },

    removePost(post) {
      this.posts = this.posts.filter((p) => p.id !== post.id);
    },

    showDialog() {
      this.dialogVisible = true;
    },
  },

  mounted() {
    this.fetchPosts();
  },

  computed: {
    ...mapState({
      posts: (state) => state.post.posts,
      isPostLoading: (state) => state.post.isPostLoading,
      selectedSort: (state) => state.post.selectedSort,
      searchQuery: (state) => state.post.searchQuery,
      page: (state) => state.post.page,
      limit: (state) => state.post.limit,
      totalPage: (state) => state.post.totalPage,
      sortOptions: (state) => state.post.sortOptions,
    }),
    ...mapGetters({
      sortedPosts: 'post/sortedPosts',
      sortedAndSearchPosts: 'post/sortedAndSearchPosts',
    }),
  },
};
</script>

<style scoped>
.app__btns {
  display: flex;
  justify-content: space-between;
  margin: 15px 0;
}

.page__wrapper {
  display: flex;
  margin-top: 15px;
}

.page {
  border: 1px solid black;
  padding: 10px;
}

.current-page {
  border: 4px solid teal;
}

.observer {
  height: 30px;
  background: green;
}
</style>
