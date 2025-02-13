<template>
  <footer>
    <ul>
      <li v-for="(log, index) in logs" :key="index">
        <strong>{{ log.itemName }}</strong> 경로: 
        <span v-for="(step, idx) in log.path" :key="idx">
          {{ step.location }} 도착 ({{ step.time }})
          <span v-if="idx < log.path.length - 1"> → </span>
        </span>
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