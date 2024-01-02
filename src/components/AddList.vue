<template>
  <h2>Add New List</h2>
  <form @submit.prevent="addList">
    <label for="ListName">List Name</label>
    <input type="text" v-model="newListName" required />
    <label for="newItems">Items (comma-separated):</label>
    <input type="text" v-model="newItems" />
    <button type="submit">Add List</button>
  </form>
</template>

<script>
export default {
  name: 'AddList',
  data() {
    return {
      newListName: '',
      newItems: '',
    }
  },
  methods: {
    addList() {
      if (this.newListName.trim() !== '') {
        const itemsArray = this.newItems.split(',').map((item) => item.trim());

        const newList = {
          title: this.newListName,
          items: itemsArray.filter((item) => item !== '')
        };

        this.$emit('add-List', newList);

        this.newListName = '';
        this.newItems = '';
      }
    },
  },
};



</script>