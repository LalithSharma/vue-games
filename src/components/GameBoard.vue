<template>
  <div class="game-container">
    <!-- Splash Screen -->
    <div v-if="!gameStarted" class="splash-screen">
      <h1 class="title">Tic Tac Toe Game</h1>
      <div class="level-buttons">
        <button @click="setLevel('easy')" :class="{'active': level === 'easy'}">Easy</button>
        <button @click="setLevel('normal')" :class="{'active': level === 'normal'}">Normal</button>
        <button @click="setLevel('hard')" :class="{'active': level === 'hard'}">Hard</button>
      </div>
      <button class="start-btn" @click="startGame">Start Game</button>
    </div>

    <!-- Game Board -->
    <div v-else class="board">
      <Cell v-for="(cell, index) in board" :key="index" :value="cell" @click="makeMove(index)" />
    </div>

    <!-- Winner/Draw Message -->
    <div v-if="winner || isDraw">
      <p v-if="winner">Winner: {{ winner }}</p>
      <p v-else>It's a draw!</p>
      <button @click="resetGame">Restart</button>
    </div>
  </div>
</template>

<script>
import { ref, reactive } from 'vue';
import Cell from './cell.vue';

export default {
  components: {
    Cell
  },
  setup() {
    const level = ref('easy');  // Default level is 'easy'
    const gameStarted = ref(false);  // Tracks if the game has started
    const board = reactive(Array(9).fill(null));
    const currentPlayer = ref('X');  // 'X' is the player, 'O' is the computer
    const winner = ref(null);
    const isDraw = ref(false);

    // Set the selected level
    const setLevel = (selectedLevel) => {
      level.value = selectedLevel;
    };

    // Start the game with the selected level
    const startGame = () => {
      gameStarted.value = true;
      resetGame();
    };

    // Make a move
    const makeMove = (index) => {
      if (board[index] || winner.value) return;

      // Player's move
      board[index] = currentPlayer.value;
      if (checkWinner()) return;

      // Switch to the computer's turn
      currentPlayer.value = 'O';
      computerMove();
    };

    // Computer move (Enhanced AI based on selected level)
    const computerMove = () => {
      const availableMoves = board
        .map((cell, index) => (cell === null ? index : null))
        .filter((index) => index !== null);

      let move = findBestMove(availableMoves);
      board[move] = 'O';
      if (checkWinner()) return;
      currentPlayer.value = 'X';
    };

    // Check for a winner or a draw
    const checkWinner = () => {
      const winPatterns = [
        [0, 1, 2], [3, 4, 5], [6, 7, 8],  // Horizontal
        [0, 3, 6], [1, 4, 7], [2, 5, 8],  // Vertical
        [0, 4, 8], [2, 4, 6]              // Diagonal
      ];

      for (let pattern of winPatterns) {
        const [a, b, c] = pattern;
        if (board[a] && board[a] === board[b] && board[a] === board[c]) {
          winner.value = board[a];
          return true;
        }
      }

      // Check for a draw
      if (board.every(cell => cell !== null)) {
        isDraw.value = true;
        return true;
      }

      return false;
    };

    // Find the best move for the computer based on selected level
    const findBestMove = (availableMoves) => {
      // Adjust AI difficulty based on the selected level
      if (level.value === 'hard') {
        return findBestMoveHard(availableMoves);
      } else if (level.value === 'normal') {
        return findBestMoveNormal(availableMoves);
      } else {
        return availableMoves[Math.floor(Math.random() * availableMoves.length)];
      }
    };

    // AI Logic for hard difficulty
    const findBestMoveHard = (availableMoves) => {
      // Try to win first
      let bestMove = findWinningMove('O');
      if (bestMove !== null) return bestMove;

      // Block player's winning move
      bestMove = findWinningMove('X');
      if (bestMove !== null) return bestMove;

      // Choose the best strategic move or block
      return availableMoves[Math.floor(Math.random() * availableMoves.length)];
    };

    // AI Logic for normal difficulty
    const findBestMoveNormal = (availableMoves) => {
      // Block the player's winning move or pick random
      let bestMove = findWinningMove('O');
      if (bestMove !== null) return bestMove;

      bestMove = findWinningMove('X');
      if (bestMove !== null) return bestMove;

      return availableMoves[Math.floor(Math.random() * availableMoves.length)];
    };

    // Find a winning move for a player
    const findWinningMove = (player) => {
      const winPatterns = [
        [0, 1, 2], [3, 4, 5], [6, 7, 8],  // Horizontal
        [0, 3, 6], [1, 4, 7], [2, 5, 8],  // Vertical
        [0, 4, 8], [2, 4, 6]              // Diagonal
      ];

      for (let pattern of winPatterns) {
        const [a, b, c] = pattern;
        const cells = [board[a], board[b], board[c]];

        const emptyIndex = cells.findIndex(cell => cell === null);
        if (emptyIndex !== -1 && cells.filter(cell => cell === player).length === 2) {
          return [a, b, c][emptyIndex];
        }
      }

      return null;
    };

    // Reset the game
    const resetGame = () => {
      board.splice(0, board.length, ...Array(9).fill(null));
      currentPlayer.value = 'X';
      winner.value = null;
      isDraw.value = false;
    };

    return {
      level,
      gameStarted,
      board,
      currentPlayer,
      winner,
      isDraw,
      setLevel,
      startGame,
      makeMove,
      resetGame
    };
  }
};
</script>

<style scoped>
.board {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  grid-template-rows: repeat(3, 100px);
  gap: 10px;
}

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
</style>
