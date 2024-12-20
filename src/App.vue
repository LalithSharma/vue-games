<template>
  <div id="app">
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
      Hangman
      </button>
    </div>

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
        <div class="game-container">
          <h1>Guess Word</h1>
            <h3 class="hint-text">Hint: Review the text below and fill in to the bottom, to avoid hang!</h3>
          <h1 style="color:white">{{displayedWord}}</h1>
          <Figure :wrong-count="wrongLetters.length" />
          <!-- <Word :letters="displayedWord" :correct-letters="correctLetters" /> -->
           <div style="margin-below"></div>
          <Word :letters="letters" :correct-letters="correctLetters" />
        </div>
      </div>
    </div>
  </div>
  <Popup :status="status" :word="word" @reset="reset" />
  <Notification :show="notification" />
</template>

<script>
import { computed, onMounted, ref } from 'vue'
import SnakeGame from './components/SnakeGame.vue';
import GameBoard from './components/GameBoard.vue';
import CandyBoard from './components/Candy.vue';
import './assets/style.css'
import onKeydown from './assets/onKeydown';
import Figure from './components/Figure.vue';
import WrongLetters from './components/WrongLetters.vue';
import Word from './components/Word.vue';
import Popup from './components/Popup.vue'  ;
import Notification from './components/Notification.vue';


const words = ['programming', 'vuejs', 'composition']
const randomWord = () => words[Math.floor(Math.random() * words.length)]


export default {
  components: {
    SnakeGame,
    GameBoard,
    CandyBoard,
    Figure, Word, WrongLetters, Popup, Notification
  },
  data() {
    return {
      selectedGame: '' 
    };
  },
  setup() {
    const word = ref(randomWord())
    const guessedLetters = ref([])
    const displayedWord = ref([]);
    console.log('random',word.value)

    const letters = computed(() => word.value.split(''))
    const wrongLetters = computed(() =>
      guessedLetters.value.filter(l => !letters.value.includes(l))
    )
    const correctLetters = computed(() =>
      guessedLetters.value.filter(l => letters.value.includes(l))
    )

    const updateDisplayedWord =() => {
          displayedWord.value = word.value.split('').map((letter, index) => {
            if (index === 0 || index === 2) {
              return letter;  // Reveal first and third letters
            }
            return '_';  // Placeholder for other letters
          });
        }

        console.log('displayword',displayedWord.value)
        // Call the update function on component setup
      updateDisplayedWord();

    const status = computed(() => {
      if (wrongLetters.value.length === 6) return 'lose'
      if (letters.value.every(l => correctLetters.value.includes(l)))
        return 'win'
      return ''
    })
    const reset = () => {
      guessedLetters.value = []
      word.value = randomWord()
      updateDisplayedWord() 
    }

    const notification = ref(false)
    const showNotification = () => {
      notification.value = true
      setTimeout(() => (notification.value = false), 2000)
    }

    onKeydown(event => {
      const letter = event.key.toLowerCase()
      if (event.keyCode < 65 || event.keyCode > 90) return
      if (status.value) return
      if (guessedLetters.value.includes(letter)) {
        showNotification()
        return
      }
      guessedLetters.value.push(letter)
      letters.value.forEach((l, index) => {
        if (l === letter) {
          displayedWord.value[index] = letter
        }
      })
    })

    onMounted(()=>{
      updateDisplayedWord();
    });

    return {
      letters,
      word,
      wrongLetters,
      correctLetters,
      guessedLetters,
      notification,
      displayedWord,
      updateDisplayedWord,
      status,
      reset
    }
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

/* Styling for the <h3> tag within the <i> tag */
.hint-text {
  font-style: italic;
  color: #333; /* Optional: dark text color */
  font-size: 20px; /* Set font size */
  color: #FF6347; /* Tomato color for emphasis */
  text-align: center; /* Center align the text */
  font-weight: 400; /* Make the text bold */
  margin: 20px 0; /* Add some spacing around the text */
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
  width: max-content;
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
