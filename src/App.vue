<template>
  <div>
    <MainHeader></MainHeader>
    <div v-if="selectedList">
      <button @click="selectedList = null">Back to Shopping Lists</button>
      <h2>{{ selectedList.title }}</h2>
      <ShoppingList :shoppingLists="[selectedList]" @edit-list="editList" @delete-list="deleteList"></ShoppingList>
    </div>
    <div v-else>
      <h2>Shopping Lists</h2>
      <EditList v-if="editedList" :editedList="editedList" @save-changes="saveListChanges">
      </EditList>
      <AddList v-if="!editedList" @add-list="addNewList"></AddList>
      <table>
        <thead>
          <tr>
            <th>List Name</th>
            <th>Number of Items</th>
            <th>Edit List</th>
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
              <button @click.stop="editList(shoppingList)">Edit</button>
            </td>
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
import EditList from './components/EditList.vue'

export default {
  name: 'App',
  components: {
    MainHeader,
    ShoppingList,
    AddList,
    EditList
  },
  data() {
    return {
      shoppingLists: [],
      selectedList: null,
      editedList: null
    }
  },
  computed: {
    editedListItems: {
      get() {
        return this.editedList.items.join(', ')
      },
      set(value) {
        // Update the items array when the input value changes
        this.$set(
          this.editedList,
          'items',
          value.split(',').map((item) => item.trim())
        )
      }
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
    editList(list) {
      this.editedList = {
        id: list.id,
        title: list.title,
        items: [...list.items], // Create a copy of the items array
        updatedAt: list.updatedAt
      }
    },
    saveListChanges(updatedList) {
      // Transform items to the desired format
      const formattedItems = updatedList.items.map((item) => {
        if (typeof item === 'string') {
          // Handle the case where the item is a string (old format)
          return {
            id: Date.now(),
            'item-name': item,
            purchased: false
          }
        } else {
          // Handle the case where the item is an object (new format)
          return {
            id: item.id || Date.now(),
            'item-name': item['item-name'] || '',
            purchased: item.purchased || false
          }
        }
      })

      // Update the items in the updatedList
      updatedList.items = formattedItems

      // Find the index of the original list in the array
      const index = this.shoppingLists.findIndex((list) => list.id === updatedList.id)

      // Update the list in the array
      this.shoppingLists[index] = { ...updatedList }

      // Reset the editedList
      this.editedList = null

      // Make a PATCH request to update the list on the server
      fetch(`http://localhost:3000/lists/${updatedList.id}`, {
        method: 'PATCH',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ items: formattedItems })
      })
        .then((response) => {
          if (response.ok) {
            console.log('List updated successfully on the server.')
          } else {
            console.error('Error updating list on the server:', response.statusText)
          }
        })
        .catch((error) => {
          console.error('Error updating list on the server:', error.message)
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
