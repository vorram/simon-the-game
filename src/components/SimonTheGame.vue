<template>
  <div class="game">
    <div class="high-score">High Score: 0</div>
    <div class="score">Score: 0</div>
    <div class="game-circle">
      <div class="sector yellow" id="1"></div>
      <div class="sector blue" id="2"></div>
      <div class="sector red" id="3"></div>
      <div class="sector green" id="4"></div>
    </div>
    <button id="start" v-on:click="playCombo">START</button>
    <label for="difficulty">Difficulty:</label>
    <select id="difficulty" @input="setTimeout($event.target.value)">
      <option value="easy" selected>Easy</option>
      <option value="medium">Medium</option>
      <option value="hard">Hard</option>
    </select>
    <audio data-sound="1" src="../assets/simonSound1.mp3"></audio>
    <audio data-sound="2" src="../assets/simonSound2.mp3"></audio>
    <audio data-sound="3" src="../assets/simonSound3.mp3"></audio>
    <audio data-sound="4" src="../assets/simonSound4.mp3"></audio>
  </div>
</template>

<script>
export default {
  data() {
    return {
      score: 0,
      highScore: 0,
      generatedCombo: [],
      playerMove: 0,
      level: 0,
      controlsEnabled: false,
      timeout: 1500,
    }
  },
  methods: {
    setScore(value) {
      this.score = value
    },
    setHighScore(value) {
      this.highScore = value
    },
    setCombo(array) {
      this.generatedCombo = array
    },
    setPlayerMove(value) {
      this.playerMove = value
    },
    setLevel(value) {
      this.level = value
    },
    setTimeout(value) {
      if (value === 'easy') {
        this.timeout = 1500
      } else if (value === 'medium') {
        this.timeout = 1000
      } else if (value === 'hard') {
        this.timeout = 400
      }
    },
    generateCombo(level) {
      const getRandomInclusive = (minValue, maxValue) => {
        const min = Math.ceil(minValue)
        const max = Math.floor(maxValue)
        return Math.floor(Math.random() * (max - min + 1) + min)
      }
      const combo = []
      for (let i = 0; i < level; i++) {
        const newMove = getRandomInclusive(1, 4)
        combo.push(newMove)
      }
      return combo
    },
    lightUp(sectorNumber) {
      const sector = document.getElementById(`${sectorNumber}`)
      const audio = document.querySelector(`audio[data-sound="${sectorNumber}"]`)
      audio.currentTime = 0
      audio.play()
      sector.classList.add('playing')
      return new Promise((resolve) => {
        setTimeout(() => {
          sector.classList.remove('playing')
          resolve('done')
        }, 1500)
      })
    },
    async playCombo() {
      this.setLevel(5)
      this.setCombo(this.generateCombo(this.level))
      for (let i = 0; i < this.generatedCombo.length; i++) {
        await this.lightUp(this.generatedCombo[i])
      }
    },
    roundStart() {
      let newLevel = this.level + 1
      this.setLevel(newLevel)
    },
  },
}
</script>

<style scoped>
.game {
  width: max-content;
  margin: 20px auto;
}

.high-score,
.score {
  font-size: 2rem;
  text-align: center;
  margin-bottom: 12px;
}

.game-circle {
  width: 400px;
  height: 400px;
  border-radius: 50%;
}

.sector {
  width: 50%;
  height: 50%;
  float: left;
  border: 8px solid black;
}

.yellow {
  background: #fde047;
}

.blue {
  background: #2563eb;
  margin-left: -8px;
}

.red {
  background: #dc2626;
  margin-top: -8px;
}

.green {
  background: #16a34a;
  margin: -8px 0 0 -8px;
}

.yellow:hover,
.yellow.playing {
  background: #fef9c3;
}

.blue:hover,
.blue.playing {
  background: #60a5fa;
}

.red:hover,
.red.playing {
  background: #f87171;
}

.green:hover,
.green.playing {
  background: #4ade80;
}

label[for='difficulty'] {
  font-size: 1.3rem;
  display: block;
  text-align: center;
}

#difficulty {
  font-size: 1.3rem;
  display: block;
  margin: 16px auto;
}

#start {
  font-size: 2rem;
  padding: 8px 32px;
  display: block;
  margin: 32px auto;
  border-style: none;
  border-radius: 8px;
  background: #e11d48;
  color: #fafaf9;
}

#start:hover {
  background: #fb7185;
  cursor: pointer;
}
</style>
