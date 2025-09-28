<script lang="ts" setup>
import { computed, ref } from 'vue'

const emit = defineEmits(['store-answer', 'end-quiz'])

const props = defineProps<{
  questions: QuestionFormat[]
}>()

const currentQuestion = ref(0)
const selectedOption = ref<string | null>(null)

const shuffleOptions = computed(() => {
  const options = [...props.questions[currentQuestion.value].incorrect_answers]
  const randomIndex = Math.round(Math.random() * options.length)
  options.splice(randomIndex, 0, props.questions[currentQuestion.value].correct_answer)
  return options
})

function submitAnswer() {
  emit('store-answer', {
    question: props.questions[currentQuestion.value],
    answer: selectedOption.value,
  })

  selectedOption.value = null

  if (currentQuestion.value === props.questions.length - 1) {
    emit('end-quiz')
  } else {
    currentQuestion.value += 1
  }
}
</script>

<template>
  <section>
    <h2>Quiz {{ currentQuestion + 1 }}</h2>
    <progress :max="1" :value="(currentQuestion + 1) / props.questions.length" />

    <h3>{{ questions[currentQuestion].question }}</h3>

    <div>
      <button
        :class="{ active: option === selectedOption }"
        v-for="option in shuffleOptions"
        :key="option"
        @click="selectedOption = option"
      >
        {{ option }}
      </button>
    </div>

    <button @click="submitAnswer">Send</button>
  </section>
</template>

<style scoped>
.active {
  border: 2px solid black;
  background-color: rgb(255, 252, 206);
}
</style>
