<template>
  <div class="game-container">
    <div v-if="!gameStarted" class="splash-screen">
      <h1 class="title">Memory Card Game</h1>
      <div class="level-buttons">
        <button @click="setLevel('easy')" :class="{'active': level === 'easy'}">Easy</button>
        <button @click="setLevel('normal')" :class="{'active': level === 'normal'}">Normal</button>
        <button @click="setLevel('hard')" :class="{'active': level === 'hard'}">Hard</button>
      </div>
      <button class="start-btn" @click="startGame">Start Game</button>
    </div>

    <!-- Game Board -->
    <div v-if="gameStarted" class="game-board">
      <h1 class="title">Memory Card Game</h1>
  
      <!-- Game board -->
      <div class="board">
        <div
          v-for="(card, index) in cards"
          :key="index"
          class="card-container"
          @click="flipCard(card)"
        >
          <div
            :class="['card', {
              'flipped': card.flipped,
            }]"
          >
            <div v-if="showGrid" class="card-inner">
              <img :src="card.image" alt="Card image" class="card-image" />
            </div>
  
            <div v-if="!showGrid && card.flipped" class="card-inner">
              <img v-if="card.flipped" :src="card.image" alt="Card image" class="card-image" />
              <div v-else class="card-placeholder">
                <span class="card-placeholder-text">?</span>
              </div>
            </div>
          </div>
        </div>
      </div>
  
      <div v-if="gameOver" class="game-over">
        <h2 class="game-over-text">You Win!</h2>
        <button class="start-btn" @click="RestartGame">Exit Game</button>
      </div>

      <div v-if="gameDraw" class="game-over">
        <h2 class="game-over-text">Time Over!</h2>
        <button class="start-btn" @click="RestartGame">Restart</button>
      </div>

      <!-- Timer and Moves -->
      <div class="timer-moves">
        <div class="move-count">Score: {{ moves }}</div>
        <div class="time-count">Time: {{ timer }}s</div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';
let timeout;

export default {
  setup() {
    const cards = ref([]);
    const flippedCards = ref([]);
    const gameOver = ref(false);
    const gameDraw = ref(false);
    const moves = ref(0);
    const timer = ref(0);
    const timerInterval = ref(null);
    const showGrid = ref(true);
    const level = ref('easy');
    const gameStarted = ref(false);

    // Generate cards with images based on the level
    function generateCards() {
      const images = [
        'https://cdn.fun.paris/imageMPP/oranges.jpg', 
        'https://cdn.fun.paris/imageMPP/pomme.jpg',
        'https://cdn.fun.paris/imageMPP/kiwi.jpg',
        'https://cdn.fun.paris/imageMPP/rass.jpg',
        'https://cdn.fun.paris/imageMPP/mango.jpg',
        'https://via.placeholder.com/150/33FF57',
        'https://via.placeholder.com/150/3357FF',
        'https://via.placeholder.com/150/5733FF',
      ];
      
      // Set number of pairs based on level
      let numPairs;
      switch (level.value) {
        case 'easy':
          numPairs = 6;
          timeout = 15;
          break;
        case 'normal':
          numPairs = 12;
          timeout = 15;
          break;
        case 'hard':
          numPairs = 24;
          timeout = 5;
          break;
      }
      
      let cards = [...images.slice(0, numPairs), ...images.slice(0, numPairs)];
      cards = cards.sort(() => Math.random() - 0.5);
      return cards.map(image => ({ image, flipped: false }));
    }
    // Start timer
    function startTimer() {
      timerInterval.value = setInterval(() => {
        if (!gameOver.value) {
          timer.value += 1;
        }
      }, 1000);

      setTimeout(() => {
          showGrid.value = false; 
        }, 5000); // 5000ms = 5 seconds
    }

    // Flip a card when clicked
    function flipCard(card) {
      if (gameOver.value || card.flipped || flippedCards.value.length === 2) return;
  
      card.flipped = true;
      flippedCards.value.push(card);
      //moves.value += 1;
  
      if (flippedCards.value.length === 2) {
        checkMatch();
      }
  
      if (timer.value >= timeout) {
        gameDraw.value = true;
        showGrid.value = true; 
        clearInterval(timerInterval.value); // Stop the timer when the game ends
      }
    }

    // Check if the flipped cards match
    function checkMatch() {
      const [first, second] = flippedCards.value;
      if (first.image === second.image) {
        flippedCards.value = [];
        moves.value += 1;
        if (cards.value.every(card => card.flipped)) {
          gameOver.value = true;
          clearInterval(timerInterval.value); // Stop the timer when the game ends
        }
      } else {
        setTimeout(() => {
          first.flipped = false;
          second.flipped = false;
          flippedCards.value = [];
        }, 1000);
      }
    }

    function RestartGame(){
      //gameStarted.value = false;
      gameDraw.value = false;
      gameOver.value = false;
      timer.value = 0;
      moves.value = 0;
      clearInterval(timerInterval.value);
      startGame();
    }
    // Start the game based on the selected level
    function startGame() {
      cards.value = generateCards();
      gameStarted.value = true;
      startTimer();
    }
    // Set the level of the game
    function setLevel(selectedLevel) {
      level.value = selectedLevel;
    }

    // On before unmount
    onBeforeUnmount(() => {
      clearInterval(timerInterval.value);
    });

    return {
      cards,
      flippedCards,
      gameOver,
      gameDraw,
      moves,
      timer,
      showGrid,
      flipCard,
      startGame,
      RestartGame,
      setLevel,
      gameStarted,
      level,
    };
  }
};
</script>

<style scoped>
/* General Styling */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.game-container {  
  background-color: #f7fafc;
  display: flex;
  align-items: center;
  justify-content: center;
}

.title {
  font-size: 32px;
  font-weight: bold;
  text-align: center;
  margin-bottom: 20px;
}

/* Splash Screen Styling */
.splash-screen {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.level-buttons {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
}

.level-buttons button {
  padding: 12px 25px;
  background-color: #3182ce;
  border: none;
  border-radius: 8px;
  color: white;
  cursor: pointer;
  transition: transform 0.3s ease;
}

.level-buttons button:hover {
  transform: scale(1.05);
}

.level-buttons button.active {
  background-color: #48bb78;
}

.start-btn {
  padding: 12px 30px;
  background-color: #48bb78;
  border: none;
  border-radius: 8px;
  color: white;
  font-size: 16px;
  cursor: pointer;
  margin-top: 20px;
}

.start-btn:hover {
  background-color: #38a169;
}

/* Game Board Styling */
.game-board {
  background-color: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  padding: 24px;
  max-width: 600px;
  width: 100%;
}

.board {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
}

.card-container {
  position: relative;
  border: 1px solid lightgrey;
}

.card {
  width: 96px;
  height: 96px;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.card.flipped {
  background-color: #3182ce;
}

.card-inner {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #e2e8f0;
  border-radius: 8px;
}

.card-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 8px;
}

.card-placeholder {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #e2e8f0;
  width: 100%;
  height: 100%;
  border-radius: 8px;
}

.card-placeholder-text {
  font-size: 24px;
}

/* Game Over / Timer Styling */
.game-over {
  text-align: center;
  margin-top: 16px;
}

.game-over-text {
  font-size: 18px;
  font-weight: 600;
  color: #48bb78;
}

.timer-moves {
  display: flex;
  justify-content: space-between;
  margin-top: 16px;
  font-size: 16px;
}

.move-count, .time-count {
  color: #9b2c2c;
}
</style>
