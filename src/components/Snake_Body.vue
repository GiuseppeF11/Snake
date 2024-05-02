<template>
  <div>
    <canvas v-if="canvasReady" ref="canvas" :width="canvasSize" :height="canvasSize" class="screen m-auto"></canvas>
    <div>Punteggio: {{ score }}</div>
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
      ctx: null,
      snake: [{ x: 10, y: 10 }],
      food: { x: 15, y: 15 },
      direction: 'right',
      gameInterval: null,
      score: 0, // Inizializza il punteggio a 0
      foodsEaten: 0, // Contatore dei cibi mangiati
      initialInterval: 100 // Intervallo iniziale del gioco (in millisecondi)
    };
  },
  computed: {
    canvasSize() {
      return this.blockSize * this.numBlocksX;
    },
    currentInterval() {
      // Calcola il tempo di intervallo attuale basato sul numero di cibi mangiati
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
    startGame() {
      this.gameInterval = setInterval(this.updateGame, this.currentInterval);
    },
    updateGame() {
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
        if (confirm('HAI PERSO! Vuoi riprovare?')) {
          this.restartGame();
        } else {
          clearInterval(this.gameInterval);
        }
        return;
      }

      this.snake.unshift({ x: newX, y: newY });

      if (newX === this.food.x && newY === this.food.y) {
        this.score += 10; // Incrementa il punteggio di 10 punti ogni volta che il serpente mangia il cibo
        this.foodsEaten++; // Incrementa il contatore dei cibi mangiati
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
      this.food.x = Math.floor(Math.random() * this.numBlocksX);
      this.food.y = Math.floor(Math.random() * this.numBlocksY);
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
      switch (event.key) {
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
    },
    restartGame() {
      clearInterval(this.gameInterval);
      this.snake = [{ x: 10, y: 10 }];
      this.food = { x: 15, y: 15 };
      this.direction = 'right';
      this.score = 0; // Ripristina il punteggio a 0
      this.foodsEaten = 0; // Resetta il contatore dei cibi mangiati
      this.startGame();
    }
  },
  created() {
    window.addEventListener('keydown', this.handleKeyPress);
  },
  destroyed() {
    window.removeEventListener('keydown', this.handleKeyPress);
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
