<template>
  <div>
    <select v-model="difficulty" @change="changeDifficulty">
      <option value="easy">Facile</option>
      <option value="medium">Medio</option>
      <option value="hard">Difficile</option>
    </select>
    <canvas v-if="canvasReady" ref="canvas" :width="canvasSize" :height="canvasSize" class="screen m-auto"></canvas>
  </div>
</template>

<script>
export default {
  props: ['score'],
  data() {
    return {
      canvasReady: false,
      blockSize: 20,
      numBlocksX: 25,
      numBlocksY: 25,
      difficulty: 'easy', // Impostazione predefinita della difficoltà
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
      this.startGame();
    });
  },
  methods: {
    changeDifficulty() {
      switch (this.difficulty) {
        case 'easy':
          this.numBlocksX = 5;
          this.numBlocksY = 5;
          break;
        case 'medium':
          this.numBlocksX = 10;
          this.numBlocksY = 10;
          break;
        case 'hard':
          this.numBlocksX = 20;
          this.numBlocksY = 20;
          break;
        default:
          break;
      }
      // Riavvia il gioco con la nuova difficoltà
      this.restartGame();
    },
    startGame() {
      this.gameInterval = setInterval(this.updateGame, this.currentInterval);
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

      if (newX < 0 || newX >= this.numBlocksX || newY < 0 || newY >= this.numBlocksY || this.isCollidingWithBody(newX, newY)) {
        this.showingAlert = true;
        if (confirm('HAI PERSO! Vuoi riprovare?')) {
          this.restartGame();
        } else {
          clearInterval(this.gameInterval);
        }
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
      this.ctx.fillStyle = 'green';
      this.snake.forEach(segment => {
        this.ctx.fillRect(segment.x * this.blockSize, segment.y * this.blockSize, this.blockSize, this.blockSize);
      });
    },
    handleKeyPress(event) {
      if (event.key.startsWith('Arrow') && !this.showingAlert) {
        this.keysPressed.add(event.key);
        event.preventDefault();
      }
    },
    handleKeyUp(event) {
      if (event.key.startsWith('Arrow')) {
        this.keysPressed.delete(event.key);
      }
    },
    restartGame() {
      this.showingAlert = false;
      clearInterval(this.gameInterval);
      this.snake = [{ x: 10, y: 10 }];
      this.food = { x: 15, y: 15 };
      this.direction = 'right';
      this.score = 0;
      this.foodsEaten = 0;
      this.startGame();
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
  width: 500px;
  height: 500px;
  border: 20px inset rgb(99, 99, 99);
  border-radius: 20px;
}
</style>
