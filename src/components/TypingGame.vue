<script setup>
import { ref, computed, onMounted, watch } from 'vue'

// Kategori kata berdasarkan tingkat kesulitan
const wordPools = {
  easy: ['vue', 'js', 'code', 'web', 'html', 'css', 'game', 'fast', 'type', 'key'],
  medium: ['frontend', 'reactive', 'script', 'project', 'developer', 'browser', 'function', 'variable'],
  hard: ['asynchronous', 'composition', 'framework', 'application', 'typescript', 'component']
}

const currentDifficulty = ref('easy')
const wordsList = ref(wordPools.easy)

const currentWord = ref('Pilih tingkat kesulitan & mulai!')
const userInput = ref('')
const isPlaying = ref(false)
const gameFinished = ref(false)
const score = ref(0)
const timeLeft = ref(30)
let timerInterval = null

const totalCharsTyped = ref(0)
const errorsCount = ref(0)
const scoreHistory = ref([])

onMounted(() => {
  const saved = localStorage.getItem('typing_scores')
  if (saved) {
    scoreHistory.value = JSON.parse(saved)
  }
})

// Ubah kategori kata saat tingkat kesulitan diganti
function changeDifficulty(diff) {
  if (isPlaying.value) return
  currentDifficulty.value = diff
  wordsList.value = wordPools[diff]
  currentWord.value = getRandomWord()
}

function getRandomWord() {
  const index = Math.floor(Math.random() * wordsList.value.length)
  return wordsList.value[index]
}

function startGame() {
  isPlaying.value = true
  gameFinished.value = false
  score.value = 0
  timeLeft.value = 30
  userInput.value = ''
  totalCharsTyped.value = 0
  errorsCount.value = 0
  currentWord.value = getRandomWord()

  if (timerInterval) clearInterval(timerInterval)

  timerInterval = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--
    } else {
      endGame()
    }
  }, 1000)
}

function handleInput() {
  if (!isPlaying.value) return

  // Jika kata yang diketik cocok
  if (userInput.value.trim() === currentWord.value) {
    score.value++
    totalCharsTyped.value += currentWord.value.length
    userInput.value = ''
    currentWord.value = getRandomWord()
  }
}

function endGame() {
  isPlaying.value = false
  gameFinished.value = true
  clearInterval(timerInterval)
  currentWord.value = "Waktu Habis! 🎮"

  const newRecord = {
    date: new Date().toLocaleTimeString(),
    difficulty: currentDifficulty.value.toUpperCase(),
    score: score.value,
    wpm: wpm.value,
    accuracy: accuracy.value
  }
  scoreHistory.value.unshift(newRecord)
  if (scoreHistory.value.length > 5) scoreHistory.value.pop()

  localStorage.setItem('typing_scores', JSON.stringify(scoreHistory.value))
}

const wpm = computed(() => score.value * 2)

const accuracy = computed(() => {
  const totalAttempts = totalCharsTyped.value + errorsCount.value
  if (totalAttempts === 0) return 100
  return Math.max(0, Math.round((totalCharsTyped.value / totalAttempts) * 100))
})
</script>

<template>
  <div class="typing-game">
    <h2>⌨️ Typing Speed Challenge</h2>
    
    <!-- Pilihan Kesulitan -->
    <div class="difficulty-selector" v-if="!isPlaying">
      <button 
        v-for="diff in ['easy', 'medium', 'hard']" 
        :key="diff"
        @click="changeDifficulty(diff)"
        :class="{ active: currentDifficulty === diff }"
        class="btn-diff"
      >
        {{ diff.toUpperCase() }}
      </button>
    </div>

    <!-- Bar Statistik -->
    <div class="stats-bar">
      <div class="stat-box">Waktu: <span>{{ timeLeft }}s</span></div>
      <div class="stat-box">Skor: <span>{{ score }}</span></div>
    </div>

    <!-- Tampilan Huruf dengan Validasi Warna Real-time -->
    <div class="word-display">
      <span 
        v-for="(char, index) in currentWord" 
        :key="index"
        :class="{
          'correct-char': index < userInput.length && userInput[index] === char,
          'incorrect-char': index < userInput.length && userInput[index] !== char
        }"
      >
        {{ char }}
      </span>
    </div>

    <!-- Input Pemain -->
    <input 
      v-model="userInput" 
      @input="handleInput"
      type="text" 
      placeholder="Ketik kata di atas..."
      :disabled="!isPlaying"
      autofocus
    />

    <!-- Tombol Mulai / Main Lagi -->
    <button v-if="!isPlaying" @click="startGame" class="btn-start">
      {{ gameFinished ? 'Main Lagi 🚀' : 'Mulai Game 🎯' }}
    </button>

    <!-- Kartu Hasil Akhir -->
    <div v-if="gameFinished" class="result-card">
      <h3>📊 Hasil Permainan</h3>
      <div class="result-stats">
        <p>Skor: <strong>{{ score }} kata</strong></p>
        <p>WPM: <strong>{{ wpm }}</strong></p>
        <p>Akurasi: <strong>{{ accuracy }}%</strong></p>
      </div>
    </div>

    <!-- Riwayat Skor -->
    <div v-if="scoreHistory.length > 0" class="history-section">
      <h4>📜 Riwayat Skor Terakhir</h4>
      <ul>
        <li v-for="(item, index) in scoreHistory" :key="index">
          <span class="badge">{{ item.difficulty }}</span> 
          Skor: <strong>{{ item.score }}</strong> | WPM: {{ item.wpm }} | Akurasi: {{ item.accuracy }}%
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.typing-game {
  max-width: 480px;
  margin: 2rem auto;
  padding: 2rem;
  border-radius: 16px;
  box-shadow: 0 8px 24px rgba(0,0,0,0.1);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  text-align: center;
  background: #ffffff;
}

h2 {
  margin-bottom: 1rem;
  color: #2c3e50;
}

.difficulty-selector {
  display: flex;
  justify-content: center;
  gap: 8px;
  margin-bottom: 1.5rem;
}

.btn-diff {
  padding: 6px 14px;
  border: 2px solid #ddd;
  background: #f8f9fa;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
  font-size: 0.85rem;
  transition: all 0.2s;
}

.btn-diff.active {
  background: #42b883;
  color: white;
  border-color: #42b883;
}

.stats-bar {
  display: flex;
  justify-content: space-between;
  margin-bottom: 1rem;
  padding: 0 1rem;
  font-weight: 600;
  color: #555;
}

.stats-bar span {
  color: #42b883;
  font-size: 1.1rem;
}

.word-display {
  font-size: 2.2rem;
  font-weight: bold;
  margin-bottom: 1.2rem;
  padding: 1.2rem;
  background: #f8f9fa;
  border-radius: 12px;
  letter-spacing: 2px;
  border: 2px solid #eee;
  min-height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Warna indikator ketikan real-time */
.correct-char {
  color: #27ae60;
  background: #e8f8f5;
}

.incorrect-char {
  color: #c0392b;
  background: #fadbd8;
  text-decoration: underline;
}

input {
  width: 100%;
  padding: 0.9rem;
  font-size: 1.1rem;
  box-sizing: border-box;
  margin-bottom: 1.2rem;
  border: 2px solid #ddd;
  border-radius: 8px;
  outline: none;
  transition: border-color 0.2s;
}

input:focus {
  border-color: #42b883;
}

.btn-start {
  padding: 0.85rem;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1.1rem;
  cursor: pointer;
  font-weight: bold;
  width: 100%;
  transition: background-color 0.2s;
}

.btn-start:hover {
  background-color: #35495e;
}

.result-card {
  margin-top: 1.5rem;
  padding: 1.2rem;
  background: #f0fdf4;
  border: 1px solid #bbf7d0;
  border-radius: 10px;
  text-align: left;
}

.result-card h3 {
  margin-bottom: 0.5rem;
  color: #166534;
}

.history-section {
  margin-top: 1.5rem;
  text-align: left;
  font-size: 0.85rem;
  border-top: 1px solid #eee;
  padding-top: 1rem;
}

.history-section h4 {
  margin-bottom: 0.5rem;
  color: #444;
}

.history-section ul {
  padding-left: 18px;
  margin: 0;
}

.history-section li {
  margin-bottom: 4px;
  color: #666;
}

.badge {
  background: #e2e8f0;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: bold;
  color: #334155;
}
</style>