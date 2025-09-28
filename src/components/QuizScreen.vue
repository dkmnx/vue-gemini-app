<script lang="ts" setup>
import { computed, ref } from 'vue'

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
  if (currentQuestion.value < props.questions.length - 1) currentQuestion.value++
}
</script>

<template>
  <section>
    <h2>Quiz Component</h2>

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
