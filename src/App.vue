<template>
  <div class="page">
    <Game id="game" v-if="word" :word="word"/>
    <WordEntry @select="setWord"/>
  </div>
</template>

<script>
import WordEntry from './components/WordEntry.vue'
import Game from './components/Game.vue'

export default {
  name: 'App',
  components: {
    WordEntry,
    Game
  },
  data() {
    return { word: '' };
  },
  methods: {
    setWord(word) {
      this.word = word
      const game = document.getElementById("game")
      if (game) game.scrollIntoView({ behavior: "smooth" })
    }
  }
}
</script>

<style>
:root {
  --fg-color: #333;
  --bg-color: #eee;
  --dim-color: #777;
}

@media (prefers-color-scheme: dark) {
  :root {
    --fg-color: #ccc;
    --bg-color: #222;
    --dim-color: #888;
  }
}

html, body {
  margin: 0;
  height: 100%;
  background-color: var(--bg-color);
}

#app {
  display: flex;
  flex-direction: column;
  height: 100%;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: var(--fg-color);
  align-items: center;
  justify-content: center;
}
</style>

<style scoped>
.page {
  min-height: 0;
}
</style>
