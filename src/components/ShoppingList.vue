<template>
  <div>
    <table>
      <thead>
        <tr>
          <th>List Name</th>
          <th>Item Name</th>
          <th>Purchased</th>
        </tr>
      </thead>
      <tbody>
        <template v-for="shoppingList in shoppingLists">
          <tr v-for="item in shoppingList.items" :key="item.id">
            <td v-if="item === shoppingList.items[0]" :rowspan="shoppingList.items.length">
              {{ shoppingList.title }}
            </td>
            <td>
              <label>
                <input type="checkbox" :checked="item.purchased"
                  @change="() => updatePurchasedStatus(shoppingList, item)" />
                <span :class="{ 'crossed-off': item.purchased }">{{ item['item-name'] }}</span>
              </label>
            </td>
            <td>
              <span :class="{ 'crossed-off': item.purchased }">
                {{ item.purchased ? 'Yes' : 'No' }}
              </span>
            </td>
          </tr>
        </template>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'ShoppingList',
  props: {
    shoppingLists: Array,
  },
  methods: {
    updatePurchasedStatus(shoppingList, item) {
      item.purchased = !item.purchased;

      // Make a PATCH request to update the purchased status on the server
      fetch(`http://localhost:3000/lists/${shoppingList.id}`, {
        method: 'PATCH',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(shoppingList),
      })
        .then((response) => {
          if (!response.ok) {
            console.error('Error updating purchased status:', response.statusText);
          }
        })
        .catch((error) => {
          console.error('Error updating purchased status:', error.message);
        });
    },
  },
};
</script>

<style scoped>
.crossed-off {
  text-decoration: line-through;
}
</style>