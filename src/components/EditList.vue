<template>
  <div>
    <h2>Edit List</h2>
    <form @submit.prevent="saveChanges">
      <label for="listName">List Name</label>
      <input type="text" v-model="editedListCopy.title" required />

      <label for="editedItems">Items (comma-separated):</label>
      <input type="text" v-model="editedItems" />

      <button type="submit">Save Changes</button>
    </form>
  </div>
</template>

<script>
export default {
  name: 'EditList',
  props: {
    editedList: Object,
  },
  data() {
    return {
      editedListCopy: null,
      editedItems: '',
    };
  },
  created() {
    this.initializeEditedListCopy();
  },
  watch: {
    editedList: {
      handler() {
        this.initializeEditedListCopy();
      },
      deep: true,
    },
  },
  methods: {
    initializeEditedListCopy() {
      this.editedListCopy = { ...this.editedList };

      this.editedItems = this.editedListCopy.items.map((item) => item['item-name']).join(',');
    },
    saveChanges() {
      if (this.editedListCopy) {
        this.editedListCopy.title = this.editedListCopy.title.trim();

        this.editedListCopy.items = newItemsArray.filter((items) => item !== '');

        this.$emit('save-changes', { ...this.editedListCopy });
      }
    },
  },
};
</script>

<style scoped></style>