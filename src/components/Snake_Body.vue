<template>
  <div>
    <select v-model="difficulty" @change="changeDifficulty">
      <option value="easy">Facile</option>
      <option value="medium">Medio</option>
      <option value="hard">Difficile</option>
    </select>
    <canvas v-if="canvasReady" ref="canvas" :width="canvasSize" :height="canvasSize" class="screen m-auto"></canvas>
    <div v-if="showingAlert || backgroundBlocked" class="overlay" @click="restartGame">
      <div class="message">
        <p v-if="isGameWon" class="won_message">HAI VINTO!</p>
        <p v-else class="lose_message">HAI PERSO!</p>
        <button class="btn btn-success w-25 mx-auto">Riprova</button>
      </div>
    </div>
    <button v-if="!isGameRunning" @click="startGame" class="play-button">PLAY</button>
  </div>
</template>

<script>
export default {
  props: ['score'],
  data() {
    return {
      canvasReady: false,
      blockSize: 20,
      numBlocksX: 1,
      numBlocksY: 1,
      difficulty: 'medium',
      difficulties: {
        easy: { numBlocksX: 5, numBlocksY: 5},
        medium: { numBlocksX: 10, numBlocksY: 10},
        hard: { numBlocksX: 20, numBlocksY: 20 }
      },
      ctx: null,
      snake: [{ x: 10, y: 10 }],
      food: { x: 15, y: 15 },
      direction: 'right',
      gameInterval: null,
      score: 0,
      foodsEaten: 0,
      initialInterval: 100,
      keysPressed: new Set(),
      showingAlert: false,
      isGameWon: false,
      backgroundBlocked: false,
      isGameRunning: false
    };
  },
  computed: {
    canvasSize() {
      return this.blockSize * this.numBlocksX;
    },
    currentInterval() {
      return this.initialInterval / Math.pow(2, Math.floor(this.foodsEaten / 5));
    }
  },
  mounted() {
    this.canvasReady = true;
    this.$nextTick(() => {
      const canvas = this.$refs.canvas;
      if (!canvas) {
        console.error("Canvas non trovato.");
        return;
      }
      this.ctx = canvas.getContext('2d');
    });
  },
  methods: {
    changeDifficulty() {
      this.numBlocksX = this.difficulties[this.difficulty].numBlocksX;
      this.numBlocksY = this.difficulties[this.difficulty].numBlocksY;
      this.restartGame();
    },
    startGame() {
      this.isGameRunning = true;
      this.gameInterval = setInterval(this.updateGame, this.currentInterval);

      // Aggiorna le dimensioni del campo e la posizione di partenza
      this.numBlocksX = this.difficulties[this.difficulty].numBlocksX;
      this.numBlocksY = this.difficulties[this.difficulty].numBlocksY;
      this.snake = [{ x: Math.floor(this.numBlocksX / 2), y: Math.floor(this.numBlocksY / 2) }];
      
      // Aggiorna la posizione del cibo
      this.food = {
        x: Math.floor(Math.random() * this.numBlocksX),
        y: Math.floor(Math.random() * this.numBlocksY)
      };
    },
    updateGame() {
      const lastPressedKey = Array.from(this.keysPressed).pop();
      if (lastPressedKey) {
        switch (lastPressedKey) {
          case 'ArrowUp':
            if (this.direction !== 'down' || this.snake.length === 1) {
              this.direction = 'up';
            }
            break;
          case 'ArrowDown':
            if (this.direction !== 'up' || this.snake.length === 1) {
              this.direction = 'down';
            }
            break;
          case 'ArrowLeft':
            if (this.direction !== 'right' || this.snake.length === 1) {
              this.direction = 'left';
            }
            break;
          case 'ArrowRight':
            if (this.direction !== 'left' || this.snake.length === 1) {
              this.direction = 'right';
            }
            break;
        }
      }
      this.moveSnake();
      this.draw();

      /* if (this.foodsEaten % 3 === 0 && this.foodsEaten !== 0) {
        clearInterval(this.gameInterval);
        this.gameInterval = setInterval(this.updateGame, this.currentInterval);
      } */
    },
    moveSnake() {
      let newX = this.snake[0].x;
      let newY = this.snake[0].y;
      switch (this.direction) {
        case 'up':
          newY--;
          break;
        case 'down':
          newY++;
          break;
        case 'left':
          newX--;
          break;
        case 'right':
          newX++;
          break;
      }

      // Controlliamo se lo snake ha vinto
      if (this.snake.length === this.numBlocksX * this.numBlocksY) {
        this.isGameWon = true;
        this.backgroundBlocked = true;
        clearInterval(this.gameInterval);
        return;
      }

      if (newX < 0 || newX >= this.numBlocksX || newY < 0 || newY >= this.numBlocksY || this.isCollidingWithBody(newX, newY)) {
        this.showingAlert = true;
        this.backgroundBlocked = true;
        clearInterval(this.gameInterval);
        return;
      }

      this.snake.unshift({ x: newX, y: newY });

      if (newX === this.food.x && newY === this.food.y) {
        this.score += 10;
        this.foodsEaten++;
        this.generateFood();
      } else {
        this.snake.pop();
      }
    },
    isCollidingWithBody(x, y) {
      for (let i = 1; i < this.snake.length; i++) {
        if (this.snake[i].x === x && this.snake[i].y === y) {
          return true;
        }
      }
      return false;
    },
    generateFood() {
      let newX, newY;
      do {
        newX = Math.floor(Math.random() * this.numBlocksX);
        newY = Math.floor(Math.random() * this.numBlocksY);
      } while (this.isCollidingWithSnake(newX, newY));
      
      this.food.x = newX;
      this.food.y = newY;
    },
    isCollidingWithSnake(x, y) {
      for (let i = 0; i < this.snake.length; i++) {
        if (this.snake[i].x === x && this.snake[i].y === y) {
          return true;
        }
      }
      return false;
    },
    draw() {
      this.ctx.clearRect(0, 0, this.canvasSize, this.canvasSize);
      this.ctx.fillStyle = 'red';
      this.ctx.fillRect(this.food.x * this.blockSize, this.food.y * this.blockSize, this.blockSize, this.blockSize);
      this.ctx.fillStyle = '#4CAF50';
      this.snake.forEach(segment => {
        this.ctx.fillRect(segment.x * this.blockSize, segment.y * this.blockSize, this.blockSize, this.blockSize);
      });
    },
    handleKeyPress(event) {
      if (event.key.startsWith('Arrow') && !this.showingAlert && this.isGameRunning) {
        this.keysPressed.add(event.key);
        event.preventDefault();
      }
    },
    handleKeyUp(event) {
      if (event.key.startsWith('Arrow') && this.isGameRunning) {
        this.keysPressed.delete(event.key);
      }
    },
    restartGame() {
    this.showingAlert = false;
    this.isGameWon = false;
    this.backgroundBlocked = false;
    this.isGameRunning = false;
    clearInterval(this.gameInterval);
    
    // Aggiorna le dimensioni del campo e la posizione di partenza
    this.numBlocksX = this.difficulties[this.difficulty].numBlocksX;
    this.numBlocksY = this.difficulties[this.difficulty].numBlocksY;
    this.snake = [{ x: Math.floor(this.numBlocksX / 2), y: Math.floor(this.numBlocksY / 2) }];
    
    // Aggiorna la posizione del cibo
    this.food = {
      x: Math.floor(Math.random() * this.numBlocksX),
      y: Math.floor(Math.random() * this.numBlocksY)
    };
  
  this.direction = 'right';
  this.score = 0;
  this.foodsEaten = 0;
}

  },
  created() {
    window.addEventListener('keydown', this.handleKeyPress);
    window.addEventListener('keyup', this.handleKeyUp);
  },
  destroyed() {
    window.removeEventListener('keydown', this.handleKeyPress);
    window.removeEventListener('keyup', this.handleKeyUp);
  }
};
</script>

<style scoped>
.screen {
  background-color: rgb(216, 216, 216);
  width: 450px;
  height: 450px;
  border: 10px solid rgb(99, 99, 99);
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.message {
  display: flex;
  text-align: center;
  flex-direction: column;
  justify-content: center;
  background-image: url(https://content.imageresizer.com/images/memes/Squidward-cleaning-loser-meme-2.jpg);
  background-size: cover;
  min-width: 400px;
  min-height: 200px;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.message p {
  margin-bottom: 10px;
}

.play-button {
  background-color: #d5d5d5;
  border: none;
  color: black;
  padding: 15px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 20px; /* Ho modificato la dimensione del font qui */
  margin: 0 auto; /* Questa regola centrerà orizzontalmente il pulsante */
  transition-duration: 0.4s;
  cursor: pointer;
  border-radius: 12px;
  position: absolute;
  left: 50%; /* Posiziona il pulsante al 50% rispetto al bordo sinistro del suo contenitore */
  top: 50%; /* Posiziona il pulsante al 50% rispetto al bordo superiore del suo contenitore */
  transform: translate(-50%, -50%); /* Sposta il pulsante all'indietro del 50% della sua larghezza e altezza rispetto al suo centro */
  font-weight: 700;
}


.play-button:hover {
  color: white;
  background-color: #45a049; /* Darker Green */
}


.lose_message {
  font-size: 30px;
  font-weight: 700;
  color: antiquewhite;
  text-shadow: 0 0 20px black;
}
</style>
