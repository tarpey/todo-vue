<template>
  <div class="full-w">
    <form @submit.prevent="addTodo">
      <div class="my-5 grid grid-cols-3 gap-5">
        <label
          for="itemName"
          class="block font-semibold mb-1 opacity-0 absolute"
          >Item</label
        >
        <input
          id="itemName"
          autocomplete="off"
          name="itemName"
          class="col-span-2 bg-white w-full p-3 rounded-lg shadow-sm border-2 border-transparent focus:border-yellow-500 focus:outline-none focus:shadow-outline text-gray-600 font-medium"
          :class="{ 'border-red-500': error }"
          type="text"
          placeholder="New item..."
          ref="itemName"
          v-model="itemName"
          @change="error = ''"
        />
        <button
          :disabled="!itemName.length"
          class="transition-all bg-blue-800 font-bold text-white rounded-lg border-2 border-transparent focus:border-yellow-500 focus:outline-none focus:shadow-outline disabled:opacity-50"
        >
          Add item
        </button>
        <p v-if="error" class="text-red-500">{{ error }}</p>
      </div>
    </form>
  </div>
</template>

<script>
import GET_ITEMS from "../graphql/queries/getItems.gql";
import ADD_ITEM from "../graphql/mutations/addItem.gql";
export default {
  data() {
    return {
      itemName: "",
      error: "",
    };
  },
  methods: {
    addTodo() {
      if (!this.itemName || this.itemName.trim() === "") {
        this.error = "You need to enter an item";
        return false;
      } else {
        const name = this.itemName;
        this.$apollo
          .mutate({
            mutation: ADD_ITEM,
            variables: {
              name,
            },
            update: (cache, { data: { insert_items } }) => {
              const data = cache.readQuery({
                query: GET_ITEMS,
              });
              data.items.unshift(insert_items.returning[0]);
              cache.writeQuery({
                query: GET_ITEMS,
                data,
              });
              this.itemName = "";
              this.error = "";
              this.$refs.itemName.focus();
            },
          })
          .catch((error) => {
            console.error(error);
          });
      }
    },
  },
};
</script>
