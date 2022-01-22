<template>
  <div class="word-entry">
    <form @submit.prevent="$emit('select', word)">
      <input type="text" v-model="word" pattern="[a-z]+" required>
      <input type="submit">
    </form>
  </div>
</template>

<script>
export default {
  name: 'WordEntry',
  emits: ['select'],
  data() {
    return { word: '' }
  }
}
</script>

<style scoped>
.word-entry {
  margin: 1em;
}
</style>