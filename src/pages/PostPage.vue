<template>
  <div>
    <h1>Страница с аниме постами</h1>
    <my-input v-focus v-model="searchQuery" placeholder="Поиск..." />
    <div class="app__btns">
      <my-button @click="showDialog">Создать аниме пост</my-button>
      <my-select :options="sortOptions" v-model="selectedSort" />
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

    <!-- кастомный observer(intersection) в папке directives -->
    <div v-intersection="loadMorePosts" class="observer"></div>

    <!-- Стандартный observer -->
    <!-- <div ref="observer" class="observer"></div> -->

    <!-- Постраничный вывод 100 постов через переключения страниц цыфрами от 1-10 -->
    <!-- <div class="page__wrapper">
      <div
        v-for="pageNumber in totalPage"
        :key="pageNumber"
        class="page"
        :class="{ 'current-page': page === pageNumber }"
        @click="changePage(pageNumber)"
      >
        {{ pageNumber }}
      </div>
    </div> -->
  </div>
</template>

<script>
import PostForm from '@/components/PostForm';
import PostList from '@/components/PostList';
import axios from 'axios';

export default {
  components: {
    PostForm,
    PostList,
  },

  data() {
    return {
      posts: [],
      dialogVisible: false,
      isPostLoading: false,
      selectedSort: '',
      searchQuery: '',
      page: 1,
      limit: 10,
      totalPage: 0,
      sortOptions: [
        { value: 'title', name: 'По названию' },
        { value: 'body', name: 'По описанию' },
      ],
    };
  },

  methods: {
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

    // меняет модель page на номер страницы (постраничный вывод по страницым 1-10)
    // changePage(pageNumber) {
    //   this.page = pageNumber;
    //   // this.fetchPosts(); можно сделать вот так, лучше делать через watch. Вывод постов постранично
    // },

    async fetchPosts() {
      try {
        this.isPostLoading = true;
        const response = await axios.get(
          'https://jsonplaceholder.typicode.com/posts',
          {
            params: {
              _page: this.page,
              _limit: this.limit,
            },
          }
        );
        this.totalPage = Math.ceil(
          response.headers['x-total-count'] / this.limit
        );
        this.posts = response.data;
      } catch (error) {
        alert('ошибка');
      } finally {
        this.isPostLoading = false;
      }
    },

    async loadMorePosts() {
      try {
        this.page += 1;
        const response = await axios.get(
          'https://jsonplaceholder.typicode.com/posts',
          {
            params: {
              _page: this.page,
              _limit: this.limit,
            },
          }
        );
        this.totalPage = Math.ceil(
          response.headers['x-total-count'] / this.limit
        );
        this.posts = [...this.posts, ...response.data];
      } catch (error) {
        alert('ошибка');
      }
    },
  },

  mounted() {
    this.fetchPosts();
    // const options = {
    //   rootMargin: '0px',
    //   threshold: 1.0,
    // };
    // const callback = (entries, observer) => {
    //   if (entries[0].isIntersecting && this.page < this.totalPage) {
    //     this.loadMorePosts();
    //   }
    // };
    // const observer = new IntersectionObserver(callback, options);
    // observer.observe(this.$refs.observer);
  },

  computed: {
    sortedPosts() {
      return [...this.posts].sort((post1, post2) => {
        return post1[this.selectedSort]?.localeCompare(
          post2[this.selectedSort]
        );
      });
    },

    sortedAndSearchPosts() {
      return this.sortedPosts.filter((post) =>
        post.title.toLowerCase().includes(this.searchQuery.toLowerCase())
      );
    },
  },

  // watch: {
  //   page() {
  //     this.fetchPosts();
  //   },
  // },

  // способ сортировки через watch(аналог useEffect), выше этот способ через computed
  // watch: {
  //   selectedSort(newValue) {
  //     this.posts.sort((post1, post2) => {
  //       return post1[newValue]?.localeCompare(post2[newValue]);
  //     });
  //   },
  // },
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
