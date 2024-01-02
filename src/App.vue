<template>
  <div>
    <MainHeader></MainHeader>
    <div v-if="selectedList">
      <button @click="selectedList = null">Back to Shopping Lists</button>
      <h2>{{ selectedList.title }}</h2>
      <ShoppingList :shoppingLists="[selectedList]" @delete-list="deleteList"></ShoppingList>
    </div>
    <div v-else>
      <AddList @add-list="addNewList"></AddList>
      <table>
        <thead>
          <tr>
            <th>List Name</th>
            <th>Number of Items</th>
            <th>Delete List</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="shoppingList in shoppingLists" :key="shoppingList.id">
            <td>
              <a href="#" @click.prevent="showItems(shoppingList)">{{ shoppingList.title }}</a>
            </td>
            <td>{{ shoppingList.items.length }}</td>
            <td>
              <button @click.stop="deleteList(shoppingList.id)">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import MainHeader from './components/MainHeader.vue'
import ShoppingList from './components/ShoppingList.vue'
import AddList from './components/AddList.vue'

export default {
  name: 'App',
  components: {
    MainHeader,
    ShoppingList,
    AddList,
  },
  data() {
    return {
      shoppingLists: [],
      selectedList: null,
    }
  },
  mounted() {
    this.fetchData()
  },
  methods: {
    fetchData() {
      fetch('http://localhost:3000/lists/')
        .then((response) => response.json())
        .then((data) => {
          this.shoppingLists = data
        })
        .catch((error) => {
          console.error('Error fetching shopping lists:', error)
        })
    },
    showItems(list) {
      this.selectedList = list
    },
    addNewList(newList) {
      const newListWithPurchasedStatus = {
        id: Date.now(),
        title: newList.title,
        items: newList.items.map((itemName, index) => ({
          id: index + 1,
          'item-name': itemName,
          purchased: false
        })),
        updatedAt: new Date().toISOString()
      }

      this.shoppingLists.push(newListWithPurchasedStatus)
      this.postNewList(newListWithPurchasedStatus)
    },
    postNewList(newList) {
      fetch('http://localhost:3000/lists/', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(newList)
      })
        .then((response) => {
          if (response.ok) {
            console.log('List added successfully.')
          } else {
            console.error('Error adding list:', response.statusText)
          }
        })
        .catch((error) => {
          console.error('Error adding list:', error.message)
        })
    },
    deleteList(listId) {
      if (confirm('Are you sure you want to delete this list?')) {
        fetch(`http://localhost:3000/lists/${listId}`, {
          method: 'DELETE'
        })
          .then((response) => {
            if (response.ok) {
              console.log('List deleted successfully.')
              this.shoppingLists = this.shoppingLists.filter((list) => list.id !== listId)
            } else {
              console.error('Error deleting list:', response.statusText)
            }
          })
          .catch((error) => {
            console.error('Error deleting list:', error.message)
          })
      }
    }
  }
}
</script>

<style>
.crossed-off {
  text-decoration: line-through;
}
</style>