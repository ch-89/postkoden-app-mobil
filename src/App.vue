<script setup>
import data from "@/data.json"
import { computed, ref } from "vue"

const questions = ref(null)
const index = ref(0)
const attempts = ref(1)
const selected = ref([])
const status = ref(null)
const hint = ref(false)
let subject

const prices = [
  1000,
  2000,
  3000,
  5000,
  10000,
  15000,
  25000,
  50000,
  75000,
  100000,
  150000,
  250000,
  400000,
  500000,
  1000000
]

const helpers = ref([
  { name: "50:50", icon: "fa-circle-half-stroke", used: false, callback: fiftyFifty },
  { name: "2 attempts", icon: "fa-shield", used: false, callback: () => attempts.value++ },
  { name: "Hint", icon: "fa-lightbulb", used: false, callback: () => hint.value = true },
  { name: "Change Question", icon: "fa-rotate", used: false, callback: changeQuestion },
])

const question = computed(() => questions.value[index.value])

const start = () => {
  // const difficulties = [
  //   { level: "easy", count: 5 },
  //   { level: "medium", count: 5 },
  //   { level: "hard", count: 4 },
  //   { level: "very hard", count: 1 },
  // ]
  const difficulties = [
    { level: "easy", count: 5 },
    { level: "medium", count: 5 },
    { level: "hard", count: 5 },
    // { level: "very hard", count: 1 },
  ]

  const shuffledQuestions = []

  difficulties.forEach(({ level, count }) => {
    const randomQuestions = data
      .filter(question => question.difficulty === level)
      .sort(() => Math.random() - 0.5) // Shuffle the filtered questions
      .slice(0, count) // Take the specified number of questions

    shuffledQuestions.push(...randomQuestions)
  })

  console.log(shuffledQuestions)

  shuffledQuestions.forEach(question => question.options.sort(() => Math.random() - 0.5))

  questions.value = shuffledQuestions.map((question, i) => ({
    ...question,
    price: prices[i]
  }))

  index.value = 0
  status.value = null // Reset game state

  helpers.value.forEach(helper => helper.used = false)
}

start()

const reset = () => {
  questions.value = null
  index.value = 0
  status.value = null
  helpers.value.forEach(helper => helper.used = false)
}

const select = option => {
  selected.value.push(option)

  if (selected.value.length == attempts.value || selected.value.includes(question.value.answer)) {
    setTimeout(() => {
      if (selected.value.includes(question.value.answer)) {
        if (index.value < questions.value.length - 1) {
          index.value++
        }
        else {
          status.value = "win"
          celebrate()
        }
      }
      else {
        status.value = "lose"
      }

      selected.value = []
      attempts.value = 1
      hint.value = false
    }, 5000)
  }
}

const use = helper => {
  helper.used = true

  if (question.value.helpers) {
    question.value.helpers.push(helper.icon)
  }
  else {
    question.value.helpers = [helper.icon]
  }

  helper.callback()
}

function fiftyFifty() {
  attempts.value += 2

  question.value.options
    .filter(option => option != question.value.answer && !selected.value.includes(option))
    .sort(() => Math.random() - 0.5)
    .slice(0, 2)
    .forEach(option => selected.value.push(option))
}

function changeQuestion() {
  const filteredQuestions = data.filter(
    // q => q.difficulty === question.value.difficulty && q.text !== question.value.text
    q => q.difficulty === question.value.difficulty && !questions.value.includes(q)
  )

  const randomIndex = Math.floor(Math.random() * filteredQuestions.length)
  const newQuestion = filteredQuestions[randomIndex]

  // Replace the current question with the new one
  Object.assign(questions.value[index.value], newQuestion)

  // Reset selected options and hint
  selected.value = []
  hint.value = false
}

const celebrate = () => {
  // Confetti
  // const duration = 10 * 1000 // 10 seconds
  // const end = Date.now() + duration
  // const colors = ['#26ccff', '#a25afd', '#ff5e7e', '#88ff5a', '#fcff42', '#ffa62d', '#ff36ff']

  // function randomInRange(min, max) {
  //   return Math.random() * (max - min) + min
  // }

  // (function frame() {
  //   confetti({
  //     particleCount: 1,
  //     origin: { x: Math.random(), y: 0 },
  //     gravity: randomInRange(0.6, 0.8),
  //     ticks: 800,
  //     colors: [colors[Math.floor(Math.random() * colors.length)]]
  //   })

  //   if (Date.now() < end) {
  //     requestAnimationFrame(frame)
  //   }
  // })()


  var end = Date.now() + (15 * 1000)
  var colors = ['#26ccff', '#a25afd', '#ff5e7e', '#88ff5a', '#fcff42', '#ffa62d', '#ff36ff'];

  (function frame() {
    confetti({
      particleCount: 1,
      angle: 60,
      spread: 55,
      decay: 0.95,
      ticks: 400,
      origin: { x: 0 },
      colors: [colors[Math.floor(Math.random() * colors.length)]],
    })
    confetti({
      particleCount: 1,
      angle: 120,
      spread: 55,
      decay: 0.95,
      ticks: 400,
      origin: { x: 1 },
      colors: [colors[Math.floor(Math.random() * colors.length)]]
    })

    if (Date.now() < end) {
      requestAnimationFrame(frame)
    }
  }())


  // Applause
  var audio = new Audio("/sounds/applause.wav")

  audio.play()
}
</script>

<template>
  <div class="  text-gray-600 h-dvh grid grid-rows-1 relative overflow-y-hidden">
    <div v-if="status" class=" p-8 grid place-items-center">
      <header class=" text-center">
        <h2 class=" text-4xl font-bold">{{ status == "win" ? "Congratulations!" : "Game Over" }}</h2>
        <p class=" text-2xl">
          Score:
          <span class=" text-rose-400">
            {{ status == "win" ? questions.length : index }}/{{ questions.length }}
          </span>
        </p>
      </header>
      <button @click="start" class=" bg-rose-400 text-white px-4 py-2 rounded-md font-semibold">New Game</button>
    </div>
    <div v-else class="p-8 flex flex-col justify-between">
      <div class=" flex gap-4 justify-center">
        <button v-for="helper in helpers" @click.once="use(helper)"
          :disabled="selected.length == attempts || selected.includes(question.answer) || helper.used"
          class=" text-white size-12 rounded-full shadow-md text-xl transition-colors"
          :class="helper.used ? 'bg-gray-200' : 'bg-rose-400'">
          <i class="fa-solid" :class="helper.icon"></i>
        </button>
      </div>
      <div class=" text-center grid gap-4">
        <h3 class=" text-gray-400 capitalize">{{ question.subject }}</h3>
        <p class=" text-xl font-semibold text-balance">{{ question.text }}</p>
        <div v-if="hint" class=" bg-green-100 text-green-600 p-4 rounded-lg">
          <span class=" font-semibold">Ledtråd:</span>
          {{ question.hint }}
        </div>
      </div>
      <div class=" grid gap-4">
        <button v-for="option in question.options"
          :disabled="selected.length == attempts || selected.includes(question.answer) || selected.includes(option)"
          @click="select(option)" class=" bg-gray-100 px-4 py-2 transition-colors rounded-md" :class="{
            'bg-green-400 text-white': (selected.length == attempts || selected.includes(question.answer)) && option == question.answer,
            'bg-rose-400 text-white': selected.includes(option) && option != question.answer
          }">{{ option }}</button>
      </div>
    </div>
    <div class=" bg-gray-100 p-8">
      <button popovertarget="prices" class=" bg-yellow-400 text-white px-4 py-2 grid grid-cols-3 w-full rounded-md">
        <div class=" mr-auto">{{ index + 1 }}/{{ questions.length }}</div>
        <div class=" text-center font-semibold">{{ question.price }}</div>
        <div class=" ml-auto flex items-center gap-2">
          <i v-for="helper in question.helpers" :class="helper" class="fa-solid"></i>
        </div>
      </button>
    </div>
    <button popover id="prices" popovertarget="prices"
      class=" bg-white shadow-md size-auto p-8 starting:open:translate-y-full open:translate-y-0 translate-y-full transition transition-discrete duration-1000">
      <div class="flex flex-col-reverse gap-1 h-full">
        <div v-for="(question, i) in questions" :class="{ 'bg-yellow-400': i == index }"
          class=" bg-gray-100 px-4 grid grid-cols-3 grow content-center rounded-md">
          <div class=" mr-auto" :class="i == index ? 'text-white' : 'text-gray-400'">{{ i + 1 }}</div>
          <div :class="i == index ? 'text-white' : 'text-gray-600'" class=" text-center font-semibold">
            {{ question.price }}
          </div>
          <div class=" ml-auto flex items-center gap-2" :class="i == index ? 'text-white' : 'text-rose-400'">
            <i v-for="helper in question.helpers" :class="helper" class="fa-solid"></i>
          </div>
        </div>
      </div>
    </button>
  </div>
</template>