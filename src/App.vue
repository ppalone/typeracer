<template>
  <div id="root">
    <div id="info">
      <div id="timer">
        <h1>{{ timer }}</h1>
      </div>
      <div id="wpm">
        <h1>{{ wpm }} WPM</h1>
      </div>
    </div>

    <div id="word">
      <h1>{{ word }}</h1>
    </div>

    <!-- TEXT -->
    <div id="text">
      <template v-for="(t, i) in text">
        <span
          :class="{'active': i===current, 'correct': t.answer && t.word===t.answer, 'wrong': t.answer && t.word!==t.answer }">{{
          t.word
          }}</span>
      </template>
    </div>

    <!-- input -->
    <div id="input">
      <input type="text" @keydown="input" v-model="word" ref="template">
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, reactive, computed } from "vue"

const SPACE = "Space"
const BACKSPACE = "Backspace"

const TEXT = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum."

const TEXT_HACKED = TEXT.split(" ").map(t => t.toLowerCase().replace(/[,.]/g, "")).sort(() => Math.random() > 0.5 ? 1 : -1)

const text = reactive(TEXT_HACKED.map(w => {
  return {
    word: w,
    answer: null,
  }
}))
const current = ref(0)
const word = ref("")
const template = ref(null)
const counter = ref(0)

let interval = null

onMounted(() => {
  template.value.focus()
})

onBeforeUnmount(() => {
  clearInterval(interval)
})

const format = (value) => {
  return value >= 10 ? value : "0" + value
}

const timer = computed(() => {
  let t = ""
  t += format(parseInt(counter.value / 60))
  t += ":" + format(counter.value % 60)
  return t
})

const wpm = computed(() => {
  if (counter.value === 0) return 0
  return parseInt((text.filter(w => w.answer).length * (60 / (counter.value))), 10)
})

const input = (e) => {
  if (e.code === SPACE) {
    e.preventDefault()

    // don't allow empty word
    if (word.value.length === 0) {
      return
    }

    text[current.value].answer = word.value
    word.value = ""

    if (current.value >= text.length - 1) {
      clearInterval(interval)
      interval = null
      return
    }

    current.value += 1
    return
  }

  if (e.code === BACKSPACE) return

  const ALPHA = /[a-zA-Z]/g

  if (e.key.length !== 1 || !ALPHA.test(e.key)) return e.preventDefault()

  if (!interval) {
    interval = setInterval(() => {
      counter.value += 1
    }, 1000)
  }
}
</script>

<style>
#root {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 80%;
  margin: 0 auto;
}

#text {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  font-family: "Fira Code", monospace;
  font-size: 1.25rem;
  padding: .5rem;
  background-color: rgb(250, 250, 250);
}

span {
  color: rgb(125, 125, 125);
  padding: 0.25rem;
}

.active {
  background-color: rgba(225, 225, 225, 0.75);
}

#input,
input {
  width: 100%;
  margin: 0.5rem 0;
}

input {
  font-size: 1.25rem;
  padding: 0.5rem;
  font-family: "Fira Code", monospace;
}

h1 {
  font-family: "Fira Code", monospace;
  margin: 0;
}

#word {
  height: 40px;
}

.correct {
  color: rgb(0, 220, 0);
}

.wrong {
  color: rgb(220, 0, 0);
}

#info {
  width: 100%;
  display: flex;
  justify-content: space-between;
}

#info>div {
  padding: 0.25rem;
  border: 1px dashed gray;
}
</style>