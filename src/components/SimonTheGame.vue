<template>
  <div class="game">
    <div class="high-score">High Score: {{ this.highScore }}</div>
    <div class="score">Score: {{ this.score }}</div>
    <div class="gameboard">
      <div
        class="square yellow"
        id="1"
        v-on:mouseenter="highlight($event)"
        v-on:mouseleave="removeHighlight($event)"
        v-on:click="checkMove($event)"
      ></div>
      <div
        class="square blue"
        id="2"
        v-on:mouseenter="highlight($event)"
        v-on:mouseleave="removeHighlight($event)"
        v-on:click="checkMove($event)"
      ></div>
      <div
        class="square red"
        id="3"
        v-on:mouseenter="highlight($event)"
        v-on:mouseleave="removeHighlight($event)"
        v-on:click="checkMove($event)"
      ></div>
      <div
        class="square green"
        id="4"
        v-on:mouseenter="highlight($event)"
        v-on:mouseleave="removeHighlight($event)"
        v-on:click="checkMove($event)"
      ></div>
    </div>
    <div class="message" v-show="!this.gameOver">
      <p>Round {{ this.level }}</p>
      <p v-if="!this.controlsEnabled">Playing sequence...</p>
      <p v-if="this.controlsEnabled">Enter the sequence</p>
    </div>
    <div class="game-over" v-if="this.touched && this.gameOver">GAME OVER</div>
    <button id="start" v-on:click="gameStart" v-show="this.gameOver">START</button>
    <label for="difficulty" v-show="this.gameOver">Difficulty:</label>
    <select id="difficulty" @input="setTimeout($event.target.value)" v-show="this.gameOver">
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
      currentMove: 0,
      level: 0,
      controlsEnabled: false,
      timeout: 1500,
      gameOver: true,
      touched: false,
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
    setCurrentMove(value) {
      this.currentMove = value
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
    lightUp(squareNumber) {
      const square = document.getElementById(`${squareNumber}`)
      const audio = document.querySelector(`audio[data-sound="${squareNumber}"]`)
      audio.currentTime = 0
      audio.play()
      if (!square.classList.contains('playing')) {
        square.classList.add('playing')
      }
      return new Promise((resolve) => {
        setTimeout(() => {
          square.classList.remove('playing')
          resolve('done')
        }, this.timeout)
      })
    },
    highlight(e) {
      if (this.controlsEnabled) {
        if (!e.target.classList.contains('playing')) {
          e.target.classList.add('playing')
        }
      }
    },
    removeHighlight(e) {
      if (this.controlsEnabled) {
        if (e.target.classList.contains('playing')) {
          e.target.classList.remove('playing')
        }
      }
    },
    async playCombo() {
      this.setCurrentMove(0)
      this.setCombo(this.generateCombo(this.level))
      for (let i = 0; i < this.generatedCombo.length; i++) {
        await this.lightUp(this.generatedCombo[i])
      }
      this.controlsEnabled = true
    },
    roundStart() {
      const newLevel = this.level + 1
      this.setLevel(newLevel)
      setTimeout(() => {
        this.playCombo()
      }, 2000)
    },
    gameStart() {
      if (!this.touched) {
        this.touched = true
      }
      this.gameOver = false
      this.setScore(0)
      this.setLevel(0)
      this.roundStart()
    },
    checkMove(e) {
      if (this.controlsEnabled) {
        const move = Number(e.target.id)
        this.lightUp(move)
        if (move === this.generatedCombo[this.currentMove]) {
          this.score += 1
          if (this.score > this.highScore) {
            this.setHighScore(this.score)
          }
          if (this.currentMove === this.generatedCombo.length - 1) {
            this.controlsEnabled = false
            this.roundStart()
          } else {
            this.currentMove += 1
          }
        } else {
          this.endGame()
        }
      }
    },
    endGame() {
      this.gameOver = true
      this.controlsEnabled = false
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

.gameboard {
  width: 400px;
  height: 400px;
}

.square {
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

.yellow.playing {
  background: #fefce8;
}

.blue.playing {
  background: #93c5fd;
}

.red.playing {
  background: #fca5a5;
}

.green.playing {
  background: #86efac;
}

label[for='difficulty'] {
  font-size: 1.3rem;
  display: block;
  text-align: center;
}

.message {
  font-size: 1.5em;
  margin: 20px auto;
  text-align: center;
  width: 80%;
  background: #fcd34d;
  padding: 12px 36px;
}

.game-over {
  margin: 20px auto;
  text-align: center;
  width: max-content;
  padding: 12px 36px;
  background: #e11d48;
  font-size: 2em;
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
  background: #15803d;
  color: #fafaf9;
}

#start:hover {
  background: #22c55e;
  cursor: pointer;
}
</style>
