<template>
  <div id="app">
    <!-- Menu Section -->
    <div class="menu">
      <button 
        @click="selectedGame = 'snake'" 
        :class="{'active': selectedGame === 'snake'}"
      >
        Snake Game
      </button>
      <button 
        @click="selectedGame = 'candy'" 
        :class="{'active': selectedGame === 'candy'}"
      >
        Candy Game
      </button>
      <button 
        @click="selectedGame = 'ticTacToe'" 
        :class="{'active': selectedGame === 'ticTacToe'}"
      >
        Tic-Tac-Toe
      </button>

      <button 
        @click="selectedGame = 'jigsaw'" 
        :class="{'active': selectedGame === 'jigsaw'}"
      >
        Jigsaw
      </button>
    </div>

    <!-- Game Display Section -->
    <div class="game-container">
      <div v-if="selectedGame === 'snake'" class="game-card">
        <SnakeGame />
      </div>
      <div v-if="selectedGame === 'candy'" class="game-card">
        <CandyBoard />
      </div>
      <div v-if="selectedGame === 'ticTacToe'" class="game-card">
        <GameBoard />
      </div>
      <div v-if="selectedGame === 'jigsaw'" class="game-card">
        <Jigsaw />
      </div>
    </div>
  </div>
</template>

<script>
import SnakeGame from './components/SnakeGame.vue';
import GameBoard from './components/GameBoard.vue';
import CandyBoard from './components/Candy.vue';
import Jigsaw from './components/Jigsaw.vue';

export default {
  components: {
    SnakeGame,
    GameBoard,
    CandyBoard,
    Jigsaw
  },
  data() {
    return {
      selectedGame: '' // Default game to display (change to 'candy' or 'ticTacToe' as needed)
    };
  }
};
</script>

<style scoped>
/* General reset and styling */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: 'Arial', sans-serif;
}

/* Main layout */
#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #f4f7fc;
  min-height: 100vh;
  padding: 20px;
}

/* Menu section styles */
.menu {
  display: flex;
  gap: 20px;
  margin-bottom: 30px;
}

.menu button {
  padding: 12px 25px;
  background-color: #007BFF;
  border: none;
  border-radius: 8px;
  color: white;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.2s ease;
}

.menu button:hover {
  background-color: #0056b3;
  transform: scale(1.05);
}

.menu button.active {
  background-color: #28a745;
}

.menu button:focus {
  outline: none;
}

/* Game container and card styles */
.game-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  width: 100%;
}

.game-card {
  background-color: white;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0px 6px 18px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 600px;
  transition: transform 0.3s ease;
}

.game-card:hover {
  transform: scale(1.05);
}

/* Smooth transition for component switch */
.game-container > div {
  animation: fadeIn 0.5s ease-out;
}

/* Fade-in animation for smooth transitions */
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
