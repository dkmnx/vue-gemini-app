<script setup lang="ts">
import { ref } from 'vue'
import StartScreen from './components/StartScreen.vue'
import { GoogleGenAI, Type } from '@google/genai'
import QuizScreen from './components/QuizScreen.vue'
import LoadingScreen from './components/LoadingScreen.vue'

const apiKey = import.meta.env.VITE_GEMINI_API_KEY

const question = ref<Questions | undefined>(undefined)
const status = ref('start')
const isError = ref(false)

async function startQuiz(topic: string) {
  status.value = 'loading'
  isError.value = false

  try {
    const ai = new GoogleGenAI({ apiKey })

    const config = {
      responseMimeType: 'application/json',
      responseSchema: {
        type: Type.OBJECT,
        properties: {
          response_code: {
            type: Type.NUMBER,
          },
          results: {
            type: Type.ARRAY,
            items: {
              type: Type.OBJECT,
              properties: {
                type: {
                  type: Type.STRING,
                },
                difficulty: {
                  type: Type.STRING,
                },
                category: {
                  type: Type.STRING,
                },
                question: {
                  type: Type.STRING,
                },
                correct_answer: {
                  type: Type.STRING,
                },
                incorrect_answers: {
                  type: Type.ARRAY,
                  items: {
                    type: Type.STRING,
                  },
                },
              },
              propertyOrdering: [
                'type',
                'difficulty',
                'category',
                'question',
                'correct_answer',
                'incorrect_answers',
              ],
            },
          },
        },
        propertyOrdering: ['response_code', 'results'],
      },
    }

    const contents = `
      Create 5 quiz question about ${topic}.
      Difficulty: Easy to Hard
      Type: Mutiple Choice
    `

    async function main() {
      const response = await ai.models.generateContent({
        model: 'gemini-2.5-flash',
        contents,
        config,
      })
      question.value = JSON.parse(response.text ?? '')
      status.value = 'ready'
    }

    await main()
  } catch (err) {
    console.error(err)
    status.value = 'start'
    isError.value = true
  }
}
</script>

<template>
  <h1>Vue Quiz Generator</h1>
  <StartScreen v-if="status === 'start'" @start-quiz="startQuiz" />
  <QuizScreen v-if="status === 'ready'" :questions="question!.results" />
  <LoadingScreen v-if="status === 'loading'" />
  <p v-show="isError">Something went wrong!</p>
</template>

<style scoped></style>
