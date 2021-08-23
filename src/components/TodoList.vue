<template>
  <div v-if="$apollo.queries.items.loading">
    <div class="my-20 grid justify-items-center">
      <div class="loading-spinner">
        <div></div>
        <div></div>
        <div></div>
        <div></div>
      </div>
    </div>
  </div>
  <div v-else>
    <transition-group name="item-list" tag="div">
      <div v-for="(item, index) in items" :key="item.id">
        <TodoItem v-bind:item="item" :index="index" />
      </div>
    </transition-group>
    <p v-if="this.items.length" class="text-center mt-5 text-gray-500">
      {{ completedItems }}
    </p>
  </div>
</template>

<script>
import TodoItem from "./TodoItem.vue";
import GET_ITEMS from "../graphql/queries/getItems.gql";
export default {
  computed: {
    completedItems() {
      const itemsCompleted = this.items.filter((item) => item.complete === true)
        .length;
      return `${itemsCompleted}/${this.items.length} items completed`;
    },
  },
  components: {
    TodoItem,
  },
  apollo: {
    items: GET_ITEMS,
  },
};
</script>
