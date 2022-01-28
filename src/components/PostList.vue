<template>
  <div v-if="posts.length > 0">
    <h3>Список аниме</h3>
    <transition-group name="post-list">
      <post-item
        @remove="$emit('remove', post)"
        :key="post.id"
        v-for="post in posts"
        :post="post"
      />
    </transition-group>
  </div>

  <h2 v-else style="color: red">Список аниме пуст</h2>
</template>

<script>
import PostItem from '@/components/PostItem';
export default {
  components: { PostItem },
  props: {
    posts: {
      type: Array,
      required: true,
    },
  },
};
</script>

<style scoped>
.post-list-item {
  display: inline-block;
  margin-right: 10px;
}

.post-list-enter-active,
.post-list-leave-active {
  transition: all 0.4s ease;
}

.post-list-enter-from,
.post-list-leave-to {
  opacity: 0;
  transform: translateX(130px);
}

.post-list-move {
  transition: transform 0.4s ease;
}
</style>
