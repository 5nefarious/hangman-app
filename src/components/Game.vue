<template>
  <div id="game" class="container" :style="gridStyle">
    <div class="guessed-letter grid-item" v-for="char, i in guesses"
     :key="i" :style="{ 'grid-column': i + 2 }">
     <h1>{{ char }}</h1>
    </div>
    <div class="row-label grid-item" v-for="char, i in chars"
     :key="i" :style="{ 'grid-row': i + 2 }">
      {{ char }}
    </div>
    <div class="prob-cell grid-item" v-for="{ value, index }, i in probs"
     :key="i" :style="{ 'grid-row': index[0] + 2, 'grid-column': index[1] + 2 }">
      <small>{{ displayProb(value) }}</small>
    </div>
    <div class="prob-cell grid-item" v-for="{ value, index }, i in charProbs"
     :key="i" :style="{ 'grid-row': index[0] + 2, 'grid-column': -1 }">
      <small>{{ displayProb(value) }}</small>
    </div>
  </div>
</template>

<script>
import { create, all } from 'mathjs'

const config = { }
const math = create(all, config)

export default {
  name: 'Game',
  props: {
    word: {
      type: String,
      required: true
    },
    delay: {
      type: Number,
      default: 1.0
    },
  },
  data() {
    return {
      chars: "",
      guesses: "",
      probs: math.matrix(),
    }
  },
  computed: {
    numCols() {
      return this.word.length + 2
    },
    gridStyle() {
      return { 'grid-template-columns': "1fr ".repeat(this.numCols) }
    },
    charProbs() {
      try {
        return math.multiply(this.probs, math.ones(this.word.length, 1))
      } catch (err) {
        return math.zeros(this.chars.length, 1)
      }
    }
  },
  methods: {
    resetBoard(word) {
      this.guesses = '_'.repeat(word.length)
      this.probs = math.zeros(this.chars.length, word.length)
    },
    displayProb(val) {
      return (100 * val).toFixed(1)
    }
  },
  watch: {
    word(newWord) { this.resetBoard(newWord) }
  },
  mounted() {
    this.resetBoard(this.word)
  }
}
</script>

<style scoped>
.container {
  display: grid;
}

.grid-item {
  display: flex;
  align-items: center;
  justify-content: center;
}

.row-label {
  grid-column: 1;
}

.guessed-letter {
  grid-row: 1;
}

.prob-cell {
  width: 3em;
  height: 2em;
  color: var(--dim-color);
}

#game {
  margin: 1em;
}
</style>