<template>
  <div>
    <!-- Dropdown per selezionare la difficoltà -->
    <select v-model="difficulty" @change="changeDifficulty">
      <option value="easy">Facile</option>
      <option value="medium">Medio</option>
      <option value="hard">Difficile</option>
    </select>

    <!-- Canvas per il gioco Snake, viene visualizzato solo se canvasReady è true -->
    <canvas v-if="canvasReady" ref="canvas" :width="canvasSize" :height="canvasSize" class="screen m-auto"></canvas>

    <!-- Overlay per mostrare il messaggio di vittoria/sconfitta -->
    <div v-if="showingAlert || backgroundBlocked" class="overlay" @click="restartGame">
      <div class="message":class="isGameWon ? 'win-bg' : 'lose-bg'">
        <h3 v-if="isGameWon">HAI VINTO!</h3>
        <h3 v-else>HAI PERSO!</h3>
        <h3 class="fs-4 text-decoration-underline">Record: {{ score }}</h3>
        <button class="w-50 mx-auto bg-white">Riprova</button>
      </div>
    </div>

    <!-- Pulsante per iniziare il gioco -->
    <button v-if="!isGameRunning" @click="startGame" class="play-button">Gioca</button>

    <!-- Mostra il punteggio corrente -->
    <h1 class="score">Punteggio: {{ score }}</h1>
  </div>
</template>

<script>
export default {
  data() {
    return {
      canvasReady: false,
      blockSize: 20,  // Dimensione di ogni blocco del gioco
      numBlocksX: 1,
      numBlocksY: 1,
      difficulty: 'easy',
      difficulties: {
        easy: { numBlocksX: 20, numBlocksY: 20 },
        medium: { numBlocksX: 10, numBlocksY: 10 },
        hard: { numBlocksX: 5, numBlocksY: 5 },
      },
      ctx: null,  // Contesto del canvas
      snake: [{ x: 10, y: 10 }],  // Posizione iniziale del serpente
      food: { x: 15, y: 15 },  // Posizione iniziale del cibo
      direction: 'right',  // Direzione iniziale del serpente
      gameInterval: null,
      score: 0,
      foodsEaten: 0,
      initialInterval: 100,  // Intervallo iniziale di aggiornamento del gioco
      keysPressed: new Set(),
      showingAlert: false,
      isGameWon: false,
      backgroundBlocked: false,
      isGameRunning: false
    };
  },
  computed: {
    // Calcola la dimensione del canvas in base al numero di blocchi e alla dimensione di ciascun blocco
    canvasSize() {
      return this.blockSize * this.numBlocksX;
    },
    // Calcola l'intervallo di aggiornamento del gioco in base al numero di cibi mangiati
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
      this.ctx = canvas.getContext('2d');  // Ottiene il contesto 2D del canvas
    });
  },
  methods: {
    // Cambia la difficoltà del gioco
    changeDifficulty() {
      this.numBlocksX = this.difficulties[this.difficulty].numBlocksX;
      this.numBlocksY = this.difficulties[this.difficulty].numBlocksY;
      this.restartGame();  // Riavvia il gioco con la nuova difficoltà
    },
    // Inizia il gioco
    startGame() {
      this.isGameRunning = true;
      this.gameInterval = setInterval(this.updateGame, this.currentInterval);

      // Aggiorna le dimensioni del campo e la posizione di partenza del serpente
      this.numBlocksX = this.difficulties[this.difficulty].numBlocksX;
      this.numBlocksY = this.difficulties[this.difficulty].numBlocksY;
      this.snake = [{ x: Math.floor(this.numBlocksX / 2), y: Math.floor(this.numBlocksY / 2) }];
      
      // Aggiorna la posizione del cibo
      this.food = {
        x: Math.floor(Math.random() * this.numBlocksX),
        y: Math.floor(Math.random() * this.numBlocksY)
      };
    },
    // Aggiorna lo stato del gioco
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
      this.moveSnake();  // Sposta il serpente
      this.draw();  // Disegna il serpente e il cibo

      // Incremento della difficoltà (commentato perché non utilizzato)
      /* if (this.foodsEaten % 2 === 0 && this.foodsEaten !== 0) {
        clearInterval(this.gameInterval);
        this.gameInterval = setInterval(this.updateGame, this.currentInterval);
      } */
    },
    // Muove il serpente
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

      // Controlla se il giocatore ha vinto
      if (this.snake.length === this.numBlocksX * this.numBlocksY) {
        this.isGameWon = true;
        this.backgroundBlocked = true;
        clearInterval(this.gameInterval);
        return;
      }

      // Controlla se il serpente ha colpito un muro o se stesso
      if (newX < 0 || newX >= this.numBlocksX || newY < 0 || newY >= this.numBlocksY || this.isCollidingWithBody(newX, newY)) {
        this.showingAlert = true;
        this.backgroundBlocked = true;
        clearInterval(this.gameInterval);
        return;
      }

      this.snake.unshift({ x: newX, y: newY });

      // Controlla se il serpente ha mangiato il cibo
      if (newX === this.food.x && newY === this.food.y) {
        this.score += 1;
        this.foodsEaten++;
        this.generateFood();
      } else {
        this.snake.pop();
      }
    },
    // Controlla se il serpente ha colpito se stesso
    isCollidingWithBody(x, y) {
      for (let i = 1; i < this.snake.length; i++) {
        if (this.snake[i].x === x && this.snake[i].y === y) {
          return true;
        }
      }
      return false;
    },
    // Genera una nuova posizione per il cibo
    generateFood() {
      let newX, newY;
      do {
        newX = Math.floor(Math.random() * this.numBlocksX);
        newY = Math.floor(Math.random() * this.numBlocksY);
      } while (this.isCollidingWithSnake(newX, newY));
      
      this.food.x = newX;
      this.food.y = newY;
    },
    // Controlla se il cibo è in collisione con il serpente
    isCollidingWithSnake(x, y) {
      for (let i = 0; i < this.snake.length; i++) {
        if (this.snake[i].x === x && this.snake[i].y === y) {
          return true;
        }
      }
      return false;
    },
    // Disegna il serpente e il cibo sul canvas
    draw() {
      this.ctx.clearRect(0, 0, this.canvasSize, this.canvasSize);

      // Disegna il cibo
      this.ctx.fillStyle = 'red';
      this.ctx.fillRect(this.food.x * this.blockSize, this.food.y * this.blockSize, this.blockSize, this.blockSize);

      // Disegna il serpente
      this.ctx.fillStyle = '#C5C5C5';
      this.snake.forEach((segment, index) => {
        if (index === 0) {
          // Testa del serpente: disegna il rettangolo e gli occhi
          this.ctx.fillRect(segment.x * this.blockSize, segment.y * this.blockSize, this.blockSize, this.blockSize);

          // Disegna gli occhi
          this.ctx.fillStyle = '#2C423F'; 
          const eyeSize = this.blockSize / 4;
          const eyeOffsetX = this.blockSize / 4;
          const eyeOffsetY = this.blockSize / 4;
          const eye1X = segment.x * this.blockSize + eyeOffsetX;
          const eye1Y = segment.y * this.blockSize + eyeOffsetY;
          const eye2X = segment.x * this.blockSize + this.blockSize - eyeOffsetX - eyeSize;
          const eye2Y = segment.y * this.blockSize + eyeOffsetY;
          this.ctx.fillRect(eye1X, eye1Y, eyeSize, eyeSize);
          this.ctx.fillRect(eye2X, eye2Y, eyeSize, eyeSize);
        } else {
          // Corpo del serpente
          this.ctx.fillRect(segment.x * this.blockSize, segment.y * this.blockSize, this.blockSize, this.blockSize);
        }
      });
    },
    // Gestisce la pressione dei tasti freccia
    handleKeyPress(event) {
      if (event.key.startsWith('Arrow') && !this.showingAlert && this.isGameRunning) {
        this.keysPressed.add(event.key);
        event.preventDefault();
      }
    },
    // Gestisce il rilascio dei tasti freccia
    handleKeyUp(event) {
      if (event.key.startsWith('Arrow') && this.isGameRunning) {
        this.keysPressed.delete(event.key);
      }
    },
    // Riavvia il gioco
    restartGame() {
      this.showingAlert = false;
      this.isGameWon = false;
      this.backgroundBlocked = false;
      this.isGameRunning = false;
      clearInterval(this.gameInterval);
      
      // Aggiorna le dimensioni del campo e la posizione di partenza del serpente
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
    // Aggiunge gli event listener per le pressioni dei tasti quando il componente è creato
    window.addEventListener('keydown', this.handleKeyPress);
    window.addEventListener('keyup', this.handleKeyUp);
  },
  destroyed() {
    // Rimuove gli event listener quando il componente è distrutto
    window.removeEventListener('keydown', this.handleKeyPress);
    window.removeEventListener('keyup', this.handleKeyUp);
  }
};
</script>

<style scoped>
/* Stili per il canvas del gioco */
.screen {
  background-color: #829191;
  width: 450px;
  height: 450px;
  border: 10px solid rgb(99, 99, 99);
  border-radius: 2px;
  box-shadow: 0px 0px 20px black;
}

/* Stili per l'overlay del messaggio di vittoria/sconfitta */
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

/* Stili per il messaggio di vittoria/sconfitta */
.message {
  display: flex;
  text-align: center;
  flex-direction: column;
  justify-content: center;
  background-size: cover;
  min-width: 400px;
  min-height: 200px;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

/* Stili per lo sfondo della vittoria/sconfitta */
.lose-bg {
  background-image: url(https://content.imageresizer.com/images/memes/Squidward-cleaning-loser-meme-2.jpg);
}

.win-bg {
  background-image: url(https://i.imgflip.com/95p61.jpg?a476208);
}

/* Stili per i titoli del messaggio di vittoria/sconfitta */
.message h3 {
  margin-bottom: 10px;
  font-weight: 700;
  font-size: 40px;
  color: rgb(255, 255, 255);
  text-shadow: 0px 0px 10px black;
}

/* Stili per il pulsante di gioco */
.play-button {
  position: absolute;
  left: 50%; 
  top: 53%;
  transform: translate(-50%, -50%);
  font-weight: 700;
  background-color: aliceblue;
}

/* Stili per il messaggio di sconfitta */
.lose_message {
  font-size: 30px;
  font-weight: 700;
  color: antiquewhite;
  text-shadow: 0 0 20px black;
}

/* Stili per i pulsanti */
button {
  font-size: 18px;
  letter-spacing: 2px;
  text-transform: uppercase;
  display: inline-block;
  text-align: center;
  font-weight: bold;
  padding: 0.7em 2em;
  border: 3px solid #39824D;
  border-radius: 2px;
  position: relative;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.1);
  color: #39824D;
  text-decoration: none;
  transition: 0.3s ease all;
  z-index: 1;
}

button:before {
  transition: 0.5s all ease;
  position: absolute;
  top: 0;
  left: 50%;
  right: 50%;
  bottom: 0;
  opacity: 0;
  content: '';
  background-color: #022424;
  z-index: -1;
}

button:hover, button:focus {
  color: white;
}

button:hover:before, button:focus:before {
  transition: 0.5s all ease;
  left: 0;
  right: 0;
  opacity: 1;
}

/* Stili per il selettore della difficoltà */
select {
  font-size: 18px;
  font-weight: bold;
  border-radius: 2px;
  text-align: start;
  padding: 3px 5px;
  transition: 0.2s all ease;
}

/* Stili per il punteggio */
.score {
  position: absolute;
  bottom: 1%;
  left: 50%;
  transform: translate(-50%);
}
</style>
