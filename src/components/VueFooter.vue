<template>
  <footer>
    <ul>
      <li v-for="(log, index) in logs" :key="index">
        {{ log.itemName }} - {{ log.location }} 도착: {{ log.time }}
      </li>
    </ul>
  </footer>
</template>

<script>
export default {
  props: ["logs"],
};
</script>

<style scoped>
footer {
  border-top: 1px solid #ccc;
  padding: 10px;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  margin-bottom: 5px;
  font-size: 0.9em;
  color: gray;
}
</style>