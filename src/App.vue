<template>
  <div id="app" class="m-5 md:m-20 xl:w-1/4 md:w-1/2 md:mx-auto">
    <h1 class="font-semibold text-gray-600 text-2xl mb-2">
      To do list
    </h1>
    <TodoInput />
    <TodoList />
  </div>
</template>

<script>
import TodoInput from "./components/TodoInput.vue";
import TodoList from "./components/TodoList.vue";
export default {
  components: {
    TodoList,
    TodoInput,
  },
};
</script>
