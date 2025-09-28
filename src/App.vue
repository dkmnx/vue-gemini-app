<script setup lang="ts">
import { ref } from 'vue'
import StartScreen from './components/StartScreen.vue'
import { GoogleGenAI, Type } from '@google/genai'

const apiKey = import.meta.env.VITE_GEMINI_API_KEY

const question = ref<string | undefined>(undefined)

async function startQuiz(topic: string) {
  question.value = 'Loading question...'

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
      console.log(response.text)
      question.value = response.text
    }

    await main()
  } catch (err) {
    question.value = `Something went wrong! ${err}`
  }
}
</script>

<template>
  <h1>Vue Quiz Generator</h1>
  <StartScreen @start-quiz="startQuiz" />
  <pre>{{ question }}</pre>
</template>

<style scoped></style>
