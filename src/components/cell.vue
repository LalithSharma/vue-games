<template>
    <div class="cell" @click="makeMove">
      {{ value }}
    </div>
  </template>
  
  <script>
  export default {
    props: {
      value: String,
      index: Number
    },
    methods: {
      makeMove() {
        if (!this.value) {
          this.$emit('move', this.index);
        }
      }
    }
  }
  </script>
  
  <style scoped>
  .cell {
    width: 100px;
    height: 100px;
    border: 1px solid #333;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 2rem;
    cursor: pointer;
    user-select: none;
  }
  </style>
  