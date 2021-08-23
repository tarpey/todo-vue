<template>
  <div class="p-4 mb-2 bg-white hover:shadow-sm rounded-lg">
    <div class="flex items-center">
      <input
        type="checkbox"
        :id="index"
        class="opacity-0 absolute h-8 w-8 cursor-pointer"
        v-model="item.complete"
        @click="updateTodo(item)"
      />
      <div
        class="border-2 rounded-full border-gray-400 w-6 h-6 flex flex-shrink-0 justify-center items-center mr-2"
      >
        <svg
          class="fill-current hidden w-3 h-3"
          version="1.1"
          viewBox="0 0 17 12"
          xmlns="http://www.w3.org/2000/svg"
        >
          <g fill-rule="evenodd">
            <g transform="translate(-9 -11)">
              <path
                d="m25.576 11.414c0.56558 0.55188 0.56558 1.4439 0 1.9961l-9.404 9.176c-0.28213 0.27529-0.65247 0.41385-1.0228 0.41385-0.37034 0-0.74068-0.13855-1.0228-0.41385l-4.7019-4.588c-0.56584-0.55188-0.56584-1.4442 0-1.9961 0.56558-0.55214 1.4798-0.55214 2.0456 0l3.679 3.5899 8.3812-8.1779c0.56558-0.55214 1.4798-0.55214 2.0456 0z"
              />
            </g>
          </g>
        </svg>
      </div>
      <label :for="index" class="flex-grow">
        <div v-if="item.complete">
          <del>{{ item.name }}</del>
        </div>
        <div v-else>
          {{ item.name }}
        </div>
      </label>
      <div class="flex">
        <button
          @click="deleteTodo(item, index)"
          class="rounded-lg border-2 border-transparent focus:border-yellow-500 focus:outline-none focus:shadow-outline disabled:opacity-50"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-6 w-6 cursor-pointer text-red-700 hover:text-red-900"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z"
            />
          </svg>
          <span class="hidden">Delete</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import GET_ITEMS from "../graphql/queries/getItems.gql";
import UPDATE_ITEM from "../graphql/mutations/updateItem.gql";
import DELETE_ITEM from "../graphql/mutations/deleteItem.gql";
export default {
  props: ["item", "index"],
  methods: {
    /**
     * Method to update an existing todo item
     */
    updateTodo: function(item) {
      const timestamp = new Date();
      this.$apollo
        .mutate({
          mutation: UPDATE_ITEM,
          variables: {
            id: item.id,
            complete: !item.complete,
            updated: timestamp,
          },
          update: (cache, { data: { update_items } }) => {
            const data = cache.readQuery({
              query: GET_ITEMS,
            });
            /**
             * Update the item completion and timestamp
             */
            const updatedItem = update_items.returning[0];
            updatedItem.complete = !this.item.complete;
            updatedItem.updated = timestamp;
            /**
             * Sort items by completion and then timestamp
             */
            data.items
              .sort(function(a, b) {
                return a.complete - b.complete;
              })
              .sort(function(a, b) {
                return b.updated - a.updated;
              });
            cache.writeQuery({
              query: GET_ITEMS,
              data,
            });
          },
          /**
           * Optimistic mutation results
           * Update the UI before your server responds
           */
          optimisticResponse: {
            update_items: {
              returning: [
                {
                  id: item.id,
                },
              ],
            },
          },
        })
        .catch((error) => {
          console.error(error);
        });
    },
    /**
     * Method to delete an existing todo item
     */
    deleteTodo: function(item) {
      this.$apollo.mutate({
        mutation: DELETE_ITEM,
        variables: {
          id: item.id,
        },
        update: (cache, { data: { delete_items } }) => {
          if (delete_items.affected_rows) {
            const data = cache.readQuery({
              query: GET_ITEMS,
            });
            /**
             * Update items to exclude the one deleted
             */
            data.items = data.items.filter((i) => {
              return i.id !== item.id;
            });
            cache.writeQuery({
              query: GET_ITEMS,
              data,
            });
          }
        },
        /**
         * Optimistic mutation results
         * Update the UI before your server responds
         */
        optimisticResponse: {
          delete_items: {
            affected_rows: 1,
          },
        },
      });
    },
  },
};
</script>
