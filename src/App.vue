<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'
import MemoryCard from './components/MemoryCard.vue'
import { symbols } from './symbols'
import { type Card } from './types'

const cards = ref<Card[]>([])
const moves = ref(0)
const matchedCards = ref(new Set<number>())
const openedCards = ref(new Set<number>())
const totalPairs = symbols.length
const matches = computed(() => matchedCards.value.size)
const hasTwoCardsOpened = computed(() => openedCards.value.size === 2)
const isGameWon = computed(() => matches.value === totalPairs * 2)
const CLOSE_TIMEOUT = 1000

const shuffleCards = (array: Card[]) => {
  const result = [...array]
  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[result[i], result[j]] = [result[j], result[i]]
  }
  return result
}

function resetGame() {
  moves.value = 0
  matchedCards.value.clear()
  openedCards.value.clear()
  setTimeout(() => {
    cards.value = shuffleCards([...symbols, ...symbols])
  }, CLOSE_TIMEOUT)
}

function openCard(index: number) {
  openedCards.value.add(index)
  moves.value += 1
}

function getStatus(index: number) {
  if (openedCards.value.has(index)) {
    return 'opened'
  }

  if (matchedCards.value.has(index)) {
    return 'matched'
  }

  return 'closed'
}

watch(hasTwoCardsOpened, (areTwoCardsOpened) => {
  if (!areTwoCardsOpened) return

  setTimeout(() => {
    openedCards.value.clear()
  }, CLOSE_TIMEOUT)

  const [firstIndex, secondIndex] = [...openedCards.value.values()]
  if (cards.value[firstIndex].name === cards.value[secondIndex].name) {
    matchedCards.value.add(firstIndex)
    matchedCards.value.add(secondIndex)
  }
})

onMounted(() => {
  resetGame()
})
</script>

<template>
  <div class="wrapper">
    <h1>Memory Game</h1>

    <div class="game-info">
      <div>Moves: {{ moves }}</div>
      <div>Matches: {{ matches / 2 }} / {{ totalPairs }}</div>
    </div>

    <div class="board">
      <MemoryCard
        v-for="(card, index) in cards"
        :key="index"
        :status="getStatus(index)"
        :disabled="hasTwoCardsOpened || getStatus(index) !== 'closed'"
        :image="card.image"
        @click="openCard(index)"
      />
    </div>

    <button @click="resetGame">New Game</button>

    <div v-if="isGameWon" class="win-message">Congratulations! You won in {{ moves }} moves!</div>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Arial', sans-serif;
  background-color: #f4f7f9;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  padding: 20px;
}

#app {
  width: 100%;
  max-width: 800px;
  text-align: center;
}

.wrapper {
  width: 100%;
}

h1 {
  color: #2c3e50;
  margin-bottom: 20px;
}

.game-info {
  margin-bottom: 20px;
  display: flex;
  justify-content: space-between;
  padding: 0 10px;
}

.game-info div {
  font-size: 1.2rem;
  font-weight: bold;
  color: #34495e;
}

.board {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-gap: 15px;
  margin: 0 auto;
}

button {
  margin-top: 20px;
  padding: 10px 20px;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #2980b9;
}

.win-message {
  margin-top: 20px;
  font-size: 1.5rem;
  color: #27ae60;
  font-weight: bold;
}

@media (max-width: 600px) {
  .board {
    grid-template-columns: repeat(3, 1fr);
  }
  .card {
    height: 100px;
  }
}
</style>
