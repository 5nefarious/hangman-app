<template>
  <div id="game" class="container" :style="gridStyle">
    <div class="known-letter grid-item" v-for="char, i in known"
     :key="i" :style="gridCellStyle({ col: i + 2 })">
     <h1>{{ char }}</h1>
    </div>
    <div class="row-label grid-item" v-for="char, i in chars"
     :key="i" :style="gridCellStyle({ row: i + 2 })">
      {{ char }}
    </div>
    <div class="prob-cell grid-item" v-for="{ value, index }, i in probs" :key="i"
     :style="gridCellStyle({ row: index[0] + 2, col: index[1] + 2, alpha: value })">
      <small>{{ displayProb(value) }}</small>
    </div>
    <div class="prob-cell grid-item" v-for="{ value, index }, i in charProbs" :key="i"
     :style="gridCellStyle({ row: index[0] + 2, col: -1, alpha: value })">
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
    endpoint: {
      type: String,
      default: "http://localhost:5000"
    }
  },
  data() {
    return {
      chars: "",
      known: "",
      probs: math.matrix(),
      guessed: "",
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
    },
    isKnown() {
      return this.known.split('').map((char) => char != '_')
    }
  },
  methods: {
    resetState(word) {
      this.known = '_'.repeat(word.length)
      this.probs = math.zeros(this.chars.length, word.length)
      this.guessed = ""
    },
    displayProb(val) {
      return (100 * val).toFixed(1)
    },
    async updateProbs() {
      const url = new URL(this.endpoint)
      url.searchParams.append('known', this.known)
      url.searchParams.append('excluded', this.guessed)
      const response = await fetch(url)
      const { chars, counts } = await response.json()
      if (!this.chars) this.chars = chars
      this.probs = math.zeros(this.chars.length, this.word.length)
      {
        let add = (a, b) => a + b
        let total = counts.map((row) => row.reduce(add, 0)).reduce(add, 0)
        for (let m = 0; m < counts.length; m++) {
          let char = chars[m]
          let i = this.chars.indexOf(char), j = 0
          for (let n = 0; n < counts[m].length; n++) {
            while (this.isKnown[j]) { j++ }
            this.probs.set([i, j], counts[m][n] / total)
            j++
          }
        }
      }
      this.makeGuess()
    },
    makeGuess() {
      var maxIdx = [], maxProb = 0
      for (let i = 0; i < this.chars.length; i++) {
        if (this.guessed.includes(this.chars[i])) continue
        let val = this.charProbs.get([i, 0])
        if (val == maxProb) {
          maxIdx.push(i)
        } else if (val > maxProb) {
          maxIdx = [i]
          maxProb = val
        }
      }
      var idx = math.pickRandom(maxIdx)
      var char = this.chars[idx]
      var known = this.known.split('')
      for (let i = 0; i < this.word.length; i++) {
        if (this.word[i] == char) known[i] = char
      }
      this.known = known.join('')
      this.guessed += char
      console.debug('guessed: ' + this.guessed)
      if (this.known.indexOf('_') > -1) {
        window.setTimeout(this.updateProbs, 1000 * this.delay)
      }
    },
    gridCellStyle({row, col, alpha}) {
      var style = { }
      if (row) style['grid-row'] = row
      if (col) style['grid-column'] = col
      if (alpha) {
        style['opacity'] = (alpha / 2) + 0.5
        style['background-color'] = `hsla(140, 50%, 30%, ${alpha})`
      }
      return style
    },
  },
  watch: {
    word(newWord) {
      this.resetState(newWord)
      this.updateProbs()
    }
  },
  mounted() {
    this.resetState(this.word)
    this.updateProbs()
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

.known-letter {
  grid-row: 1;
}

.prob-cell {
  width: 3em;
  height: 2em;
  opacity: 20%;
  border-radius: 0.2em;
  margin: 0.1em;
}

#game {
  margin: 1em;
}
</style>