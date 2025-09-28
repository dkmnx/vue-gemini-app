<script setup lang="ts">
import { ref } from 'vue'
import StartScreen from './components/StartScreen.vue'
import { GoogleGenAI, Type } from '@google/genai'
import QuizScreen from './components/QuizScreen.vue'
import LoadingScreen from './components/LoadingScreen.vue'
import OpenAI from 'openai'
import ResultScreen from './components/ResultScreen.vue'

type ApiProvider = 'gemini' | 'deepseek'

const apiProvider = 'gemini' as ApiProvider

let apiKey: string | undefined

switch (apiProvider) {
  case 'gemini':
    apiKey = import.meta.env.VITE_GEMINI_API_KEY
    break
  case 'deepseek':
    apiKey = import.meta.env.VITE_DEEPSEEK_API_KEY
    break
  default:
    console.log('No API Key!')
    break
}

const question = ref<Questions | undefined>(undefined)
const status = ref('start')
const isError = ref(false)
const userAnswers = ref<UserAnswer[]>([])

async function geminiMain(topic: string) {
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
}

async function deepseekMain(topic: string) {
  const openai = new OpenAI({
    baseURL: 'https://api.deepseek.com',
    apiKey,
    dangerouslyAllowBrowser: true,
  })

  async function main() {
    const systemPrompt = `
      The user will ask you to provide a topic for the questions. Please parse the answer and output them in JSON format.

      EXAMPLE INPUT:
      Create a 5 quiz question about JavaScript.
      Difficulty: Easy to Hard
      Type: Mutiple Choice

      EXAMPLE OUTPUT:
      {"response_code":0,"results":[{"type":"multiple","difficulty":"medium","category":"General Knowledge","question":"Which country drives on the left side of the road?","correct_answer":"Japan","incorrect_answers":["Germany","Russia","China"]}]}
    `

    const userPrompt = `
      Create 5 quiz question about ${topic}.
      Difficulty: Easy to Hard
      Type: Mutiple Choice
    `

    const completion = await openai.chat.completions.create({
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: userPrompt },
      ],
      model: 'deepseek-chat',
      response_format: {
        type: 'json_object',
      },
    })

    const response = completion.choices[0].message.content

    console.log(response)
    question.value = JSON.parse(response ?? '')
    status.value = 'ready'
  }

  await main()
}

async function startQuiz(topic: string) {
  status.value = 'loading'
  isError.value = false

  try {
    if (apiProvider === 'deepseek') {
      await deepseekMain(topic)
    } else {
      await geminiMain(topic)
    }
  } catch (err) {
    console.error(err)
    status.value = 'start'
    isError.value = true
  }
}

function storeAnswer(answer: UserAnswer) {
  userAnswers.value.push(answer)
}

function reset() {
  status.value = 'start'
  userAnswers.value = []
}
</script>

<template>
  <h1>Vue Quiz Generator</h1>
  <template v-if="apiKey">
    <StartScreen v-if="status === 'start'" @start-quiz="startQuiz" />
    <QuizScreen
      v-if="status === 'ready'"
      @store-answer="storeAnswer"
      @end-quiz="status = 'finished'"
      :questions="question!.results"
    />
    <LoadingScreen v-if="status === 'loading'" />
    <ResultScreen @reset="reset" v-if="status === 'finished'" :user-answers="userAnswers" />
    <p v-show="isError">Something went wrong!</p>
  </template>
  <template v-else>
    <p>No API keys!</p>
  </template>
</template>

<style scoped></style>
