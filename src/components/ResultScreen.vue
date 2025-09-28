<script lang="ts" setup>
defineProps<{ userAnswers: UserAnswer[] }>()

const emit = defineEmits(['reset'])
</script>

<template>
  <section>
    <div
      v-for="userAnswer in userAnswers"
      :key="userAnswer.answer"
      class="default"
      :class="[userAnswer.answer === userAnswer.question.correct_answer ? 'correct' : 'wrong']"
    >
      <h3>{{ userAnswer.question.question }}</h3>
      <p>{{ userAnswer.answer }}</p>
      <p class="correct-answer" v-if="userAnswer.answer !== userAnswer.question.correct_answer">
        {{ userAnswer.question.correct_answer }}
      </p>
    </div>
    <button @click="emit('reset')">Reset</button>
  </section>
</template>

<style scoped>
.default {
  margin-bottom: 5px;
  padding: 0.5rem 2rem;
}

.correct {
  border: 3px solid green;
}

.wrong {
  border: 3px solid red;
}

.correct-answer {
  color: red;
  font-style: italic;
}
</style>
