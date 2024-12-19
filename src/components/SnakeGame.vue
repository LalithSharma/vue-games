<template>
  <div class="game-container">
  <div v-if="!gameStarted" class="splash-screen">
    <h1 class="title">Snake Moves Game</h1>
    <div class="level-buttons">
      <button @click="setDifficulty('easy')" :class="{'active': level === 'easy'}">Easy</button>
      <button @click="setDifficulty('normal')" :class="{'active': level === 'normal'}">Normal</button>
      <button @click="setDifficulty('hard')" :class="{'active': level === 'hard'}">Hard</button>
    </div>
    <button class="start-btn" @click="startGame">Start Game</button>
  </div>

    <!-- Game Screen -->
    <div v-if="gameStarted" class="game-container" tabindex="0" @keydown="changeDirection">
      <div v-for="(row, rowIndex) in grid" :key="rowIndex" class="game-row">
        <div
          v-for="(cell, cellIndex) in row"
          :key="cellIndex"
          :class="['game-cell', { snake: cell === 'S', food: cell === 'F', obstacle: cell === 'O' }]">
          <!-- Food Image for Fish -->
          <img v-if="cell === 'F'" src="@/assets/diamond1.png" alt="Food" class="food-img" />
        </div>
      </div>

      <div v-if="gameOver" class="game-over">
        <h2>Game Over</h2>
        <button  @click="startNewGame">Restart</button><div></div>
        <button class="start-btn" @click="Restart">Exit</button>
      </div>

      <div v-if="gameStarted" class="score">
        <h3>Score: {{ snakeLength - 3 }}</h3>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      gridSize: 18, // Default grid size
      grid: [],
      snake: [
        { x: 10, y: 10 },
        { x: 9, y: 10 },
        { x: 8, y: 10 },
      ],
      direction: "RIGHT",
      foods: [],
      obstacles: [],
      gameOver: false,
      gameStarted: false,
      snakeLength: 3,
      interval: null,
      foodTimeouts: [],
      difficulty: 'easy', // Default difficulty
      speeds: {
        easy: 300,   // Slowest speed
        normal: 200, // Medium speed
        hard: 100,   // Fastest speed
      },
    };
  },
  
  computed: {
    grid() {
      const grid = Array.from({ length: this.gridSize }, () => Array(this.gridSize).fill(null));

      // Place the snake
      this.snake.forEach(segment => {
        grid[segment.y][segment.x] = 'S'; // 'S' represents snake body
      });

      // Place the food
      this.foods.forEach(food => {
        grid[food.y][food.x] = 'F'; // 'F' represents food
      });

      // Place the obstacles
      this.obstacles.forEach(obstacle => {
        grid[obstacle.y][obstacle.x] = 'O'; // 'O' represents obstacles
      });

      return grid;
    },
  },
  methods: {
    startGame() {
      this.gameStarted = true;
      this.startNewGame();
    },
    Restart(){
      this.gameStarted = false;
    },
    setDifficulty(level) {
      this.difficulty = level;
      if (level === 'easy') {
        this.gridSize = 20;
      } else if (level === 'normal') {
        this.gridSize = 20;
      } else if (level === 'hard') {
        this.gridSize = 20;
      }
      //this.startGame();
    },

    startNewGame() {
      this.snake = [
        { x: 10, y: 10 },
        { x: 9, y: 10 },
        { x: 8, y: 10 },
      ];
      this.direction = "RIGHT";
      this.foods = [];
      this.obstacles = [];
      this.snakeLength = 3;
      this.gameOver = false;

      this.clearFoodTimeouts();

      // Adjust speed based on difficulty
      const speed = this.speeds[this.difficulty];

      // Adjust obstacles placement based on difficulty
      this.placeObstacles();
      this.placeFood();

      if (this.interval) clearInterval(this.interval);
      this.interval = setInterval(this.moveSnake, speed);
    },

    changeDirection(event) {
      const key = event.key;
      if (key === "ArrowUp" && this.direction !== "DOWN") {
        this.direction = "UP";
      } else if (key === "ArrowDown" && this.direction !== "UP") {
        this.direction = "DOWN";
      } else if (key === "ArrowLeft" && this.direction !== "RIGHT") {
        this.direction = "LEFT";
      } else if (key === "ArrowRight" && this.direction !== "LEFT") {
        this.direction = "RIGHT";
      }
    },

    moveSnake() {
      const head = { ...this.snake[0] };

      if (this.direction === "UP") head.y -= 1;
      if (this.direction === "DOWN") head.y += 1;
      if (this.direction === "LEFT") head.x -= 1;
      if (this.direction === "RIGHT") head.x += 1;

      if (head.x < 0 || head.x >= this.gridSize || head.y < 0 || head.y >= this.gridSize) {
        this.gameOver = true;
        clearInterval(this.interval);
        return;
      }

      if (this.snake.some(segment => segment.x === head.x && segment.y === head.y)) {
        this.gameOver = true;
        clearInterval(this.interval);
        return;
      }

      if (this.obstacles.some(obstacle => obstacle.x === head.x && obstacle.y === head.y)) {
        this.gameOver = true;
        clearInterval(this.interval);
        return;
      }

      this.snake.unshift(head);

      const foodIndex = this.foods.findIndex(food => food.x === head.x && food.y === head.y);
      if (foodIndex !== -1) {
        this.snakeLength += 1;
        this.foods.splice(foodIndex, 1);
        this.placeFood();
      } else {
        this.snake.pop();
      }
    },

    placeFood() {
      let x, y;
      do {
        x = Math.floor(Math.random() * this.gridSize);
        y = Math.floor(Math.random() * this.gridSize);
      } while (this.snake.some(segment => segment.x === x && segment.y === y) || this.foods.some(food => food.x === x && food.y === y));

      this.foods.push({ x, y });
  
        // Set a timeout to remove the food after a certain time (e.g., 5 seconds)
        const timeout = setTimeout(() => {
          const foodIndex = this.foods.findIndex(food => food.x === x && food.y === y);
          if (foodIndex !== -1) {
            this.foods.splice(foodIndex, 1); // Remove the food if not collected
          }
        }, 5000); // Food time limit (5 seconds)
  
        this.foodTimeouts.push(timeout);
        //this.foods.push(newFood);

        do {
          x = Math.floor(Math.random() * this.gridSize);
          y = Math.floor(Math.random() * this.gridSize);
        } while (this.snake.some(segment => segment.x === x && segment.y === y) || this.foods.some(food => food.x === x && food.y === y));
  
        const newFoodRp = { x, y };
        this.foods.push(newFoodRp);
    },

    placeObstacles() {
      const numberOfObstacles = this.difficulty === 'easy' ? 4 : this.difficulty === 'normal' ? 6 : 10;
      for (let i = 0; i < numberOfObstacles; i++) {
        let x, y;
        do {
          x = Math.floor(Math.random() * this.gridSize);
          y = Math.floor(Math.random() * this.gridSize);
        } while (this.snake.some(segment => segment.x === x && segment.y === y) || this.obstacles.some(obstacle => obstacle.x === x && obstacle.y === y));

        this.obstacles.push({ x, y });
      }
    },

    clearFoodTimeouts() {
      this.foodTimeouts.forEach(timeout => clearTimeout(timeout));
      this.foodTimeouts = [];
    },
  },
  mounted() {
    //this.startGame(); // Trigger the start game sequence after mounting
    //this.startGame();
  },
};
</script>

<style scoped>
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  font-family: monospace;
  outline: none;
}

.game-row {
  display: flex;
}

.game-cell {
  width: 30px;
  height: 30px;
  border: 1px solid #acf5ac;
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #acf5ac;
  box-sizing: border-box;
}

.snake {
  background-color: #00ff00;
}


.food {
  background-color: transparent;
}

.food-img {
  width: 80%;
  height: 80%;
  object-fit: contain;
}

.obstacle {
  background-color: #ff0000;
}

.game-over {
  text-align: center;
  font-size: 24px;
}

.score {
  font-size: 18px;
}


.title {
  font-size: 32px;
  font-weight: bold;
  text-align: center;
  margin-bottom: 20px;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
}

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
  background-color: #0fca0f;
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
