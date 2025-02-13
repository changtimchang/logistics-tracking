<template>
  <header>
    <button @click="toggleDropdown">품목</button>
    <ul v-if="dropdownVisible" class="dropdown-menu">
      <li
        v-for="item in items"
        :key="item"
        @click="selectItem(item)"
        :class="{ selected: item === selectedItem }"
      >
        {{ item }}
      </li>
    </ul>
  </header>
</template>

<script>
export default {
  data() {
    return {
      dropdownVisible: false,
      items: ["물품1", "물품2", "물품3", "물품4", "물품5"],
      selectedItem: null,
    };
  },
  methods: {
    toggleDropdown() {
      this.dropdownVisible = !this.dropdownVisible;
    },
    selectItem(item) {
      this.selectedItem = item;
      this.$emit("item-selected", item);
      this.toggleDropdown();
    },
  },
};
</script>

<style scoped>
header {
  position: relative;
}

button {
  cursor: pointer;
}

.dropdown-menu {
  list-style-type: none;
  padding: 0;
  margin: 0;
  background-color: #f9f9f9;
  border: 1px solid #ccc;
  position: absolute;
}

.dropdown-menu li {
  padding: 10px;
  cursor: pointer;
  border-bottom: 1px solid #ddd;
}

.dropdown-menu li:last-child {
  border-bottom: none;
}

.dropdown-menu li.selected {
  color: red;
}
</style>