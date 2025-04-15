<script setup>
import List from './components/List.vue';

</script>

<template>
  <header>
  <h2> Frontend Guild meeting 18-04-2024 </h2>
  </header>

  <main>
    <List />
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;

h2 {
    color: steelblue;
    font-weight: bold;
  }
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
