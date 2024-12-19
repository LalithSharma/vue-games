<template>
  <div class="puzzle-board" tabindex="-1" @keyup.prevent="onKeyUp" @click="onClickBoard">
    <div class="puzzle-message" v-if="isTouchNeeded">Touch to start</div>
    <canvas
      ref="puzzleCanvas"
      class="puzzle-canvas"
      @click.prevent
      @mousedown.prevent
      @mouseup.prevent="onClick"
      @touchend.prevent="onTouchEnd"
      :style="canvasStyle"
      :width="internalWidth * 2"
      :height="internalHeight"
    ></canvas>
    <img v-if="isImage" :style="sourceStyle" :src="src" ref="sourceImg">
    <video
      v-else
      ref="sourceImg"
      autoplay
      loop
      playsinline
      :muted="muted"
      :src="src"
      :style="sourceStyle"
      :width="internalWidth"
      :height="internalHeight"
    >
      <source
        v-for="source in sources"
        :key="source.src"
        :src="source.src"
        :type="source.type"
      >
        No video
      </source>
    </video>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount, watch, nextTick, computed } from 'vue';
import Board from '../board.ts'
import debounce from 'lodash.debounce'
import TWEEN from '@tweenjs/tween.js';

const createBoard2D = (dx, dy) => {
  const result = [];
  let n = 0;
  for (let i = 0; i < dy; i++) {
    const sub = [];
    for (let j = 0; j < dx; j++) {
      sub.push(++n % (dx * dy));
    }
    result.push(sub);
  }
  return result;
};

const createRandomBoard2D = (dx, dy) => {
  const board2D = createBoard2D(dx, dy);
  const board = new Board(board2D);
  const methodNames = ['swapAbove', 'swapLeft', 'swapRight', 'swapBelow'];
  const len = methodNames.length;
  for (let i = 0; i < 100; i++) {
    const methodName = methodNames[Math.floor(Math.random() * len)];
    try {
      board[methodName](board.blankpos);
    } catch (e) {
      continue;
    }
  }
  return board;
};

export default {
  name: 'PuzzleBoard',
  props: {
    src: { type: String },
    sources: { type: Array },
    muted: { type: Boolean, default: true },
    animation: { type: Boolean, default: true },
    cols: { type: Number, default: 4 },
    width: { type: Number, default: 300 },
    height: { type: Number, default: 300 },
    rows: { type: Number, default: 4 },
    showNumber: { type: Boolean, default: true },
    autoResize: { type: Boolean, default: false }
  },
  setup(props, { emit }) {
    const blocks = ref([]);
    const isGoal = ref(false);
    const manhattan = ref(null);
    const hamming = ref(null);
    const internalWidth = ref(props.width);
    const internalHeight = ref(props.height);
    const board = ref(createRandomBoard2D(props.cols, props.rows));
    const rafId = ref(null);
    const _blockPositions = ref([]);
    const _isStarted = ref(false);
    const isTouchNeeded = ref(true);
    const _tmpCanvas = document.createElement('canvas');
    const _tmpCtx = _tmpCanvas.getContext('2d');
    let _lastRenderVideoTime = -1;
    let _lastRenderTime = 0;
    const puzzleBoardRef = ref(null); 
    const sourceImg = ref(null); 

    // Computed properties
    const cellWidth = computed(() => internalWidth.value / props.cols);
    const cellHeight = computed(() => internalHeight.value / props.rows);
    const isImage = computed(() => /\.(jpe?g|png|webm|gif)$/i.test(props.src));
    const canvasStyle = computed(() => ({ left: isGoal.value ? '-100%' : '0' }));
    const sourceStyle = computed(() => ({ visibility: 'hidden' }));

    // Lifecycle hooks
    onMounted(() => {
      onResize();
      window.addEventListener('resize', debounce(onResize, 300));

      if (isImage.value) {
        sourceImg.value.onload = () => {
          isTouchNeeded.value = false;
          _loadImageToCanvas();
        };
      } else {
        sourceImg.value.addEventListener('play', () => {
          isTouchNeeded.value = false;
        });
      }

      const loop = () => {
        TWEEN.update();
        const sourceImgEl = sourceImg.value;
        const canvas = puzzleBoardRef.value.querySelector('.puzzle-canvas');
        const ctx = canvas.getContext('2d');
        const w = internalWidth.value;

        if (sourceImgEl.currentTime !== _lastRenderTime) {
          _lastRenderVideoTime = sourceImgEl.currentTime;
          _loadVideoFrameToCanvas();
        }

        if (isGoal.value) {
          requestAnimationFrame(loop);
          return;
        }

        ctx.clearRect(0, 0, internalWidth.value, internalHeight.value);

        if (props.showNumber) {
          ctx.font = "24px 'Avenir', Helvetica, Arial, sans-serif";
          ctx.fillStyle = '#fafafa';
          ctx.textBaseline = 'top';
        }

        for (let i = 0, len = blocks.value.length; i < len; i++) {
          const block = blocks.value[i];
          if (block === 0) continue;
          const row = board.value.row(block - 1);
          const col = board.value.col(block - 1);
          const sourceX = cellWidth.value * col + w;
          const sourceY = cellHeight.value * row;
          const pos = _blockPositions.value[block];
          if (pos == null) continue;
          const targetX = pos.x;
          const targetY = pos.y;
          ctx.drawImage(
            canvas,
            sourceX,
            sourceY,
            cellWidth.value,
            cellHeight.value,
            targetX,
            targetY,
            cellWidth.value,
            cellHeight.value
          );
          if (props.showNumber) {
            const text = String(block);
            const margin = 5;
            ctx.strokeText(text, margin + targetX, margin + targetY);
            ctx.fillText(text, margin + targetX, margin + targetY);
          }
        }
        rafId.value = requestAnimationFrame(loop);
      };

      nextTick(loop);
      emit('init');
    });

    onBeforeUnmount(() => {
      if (rafId.value) cancelAnimationFrame(rafId.value);
    });

    // Methods
    const initBoard = () => {
      board.value = createRandomBoard2D(props.cols, props.rows);
      _blockPositions.value = [];
      _isStarted.value = false;
      emit('init');
    };

    const onResize = () => {
      if (puzzleBoardRef.value) {
        const w = puzzleBoardRef.value.offsetWidth;
        const h = puzzleBoardRef.value.offsetHeight;

        if (props.autoResize) {
          internalWidth.value = w;
          internalHeight.value = h;
        } else {
          internalWidth.value = props.width;
          internalHeight.value = props.height;
        }

        if (isImage.value) {
          nextTick(() => {
            _loadImageToCanvas();
          });
        }

        updateBlockPositions(true);
      }
    };

    // Watchers
    watch(() => props.cols, initBoard);
    watch(() => props.rows, initBoard);
    watch(() => board.value, () => {
      blocks.value = board.value.blocks;
    });
    watch(() => props.width, onResize);
    watch(() => props.height, onResize);
    watch(() => blocks.value, () => {
      const isImmediate = !props.animation;
      updateBlockPositions(isImmediate);
      isGoal.value = board.value.isGoal();
      manhattan.value = board.value.manhattan();
      hamming.value = board.value.hamming();
      emit('change', {
        blocks: blocks.value,
        isGoal: isGoal.value,
        distance: manhattan.value
      });
    });
    watch(() => isGoal.value, () => {
      if (isGoal.value) emit('finish');
    });
    watch(() => props.sources, () => {
      if (!isImage.value) {
        nextTick(() => {
          sourceImg.value.load();
          isTouchNeeded.value = true;
          sourceImg.value.play();
          sourceImg.value.addEventListener('play', () => {
            isTouchNeeded.value = false;
          });
        });
      }
    });
    watch(() => props.src, () => {
      if (isImage.value) {
        isTouchNeeded.value = false;
        nextTick(() => {
          sourceImg.value.addEventListener('load', () => {
            _loadImageToCanvas();
          });
        });
      }
    });

    const updateBlockPositions = (isImmediate) => {
      for (let i = 0, len = blocks.value.length; i < len; i++) {
        const b = blocks.value[i];
        const col = board.value.col(i);
        const row = board.value.row(i);
        const x = cellWidth.value * col;
        const y = cellHeight.value * row;
        const from = _blockPositions.value[b] || { x: 0, y: 0 };
        if (!_blockPositions.value[b]) {
          _blockPositions.value[b] = from;
        }
        if (from.x - x === 0 && from.y - y === 0) continue;
        const obj = { x: from.x, y: from.y };
        if (isImmediate) {
          _blockPositions.value[b].x = x;
          _blockPositions.value[b].y = y;
        } else {
          new TWEEN.Tween(obj)
            .to({ x, y }, 200)
            .easing(TWEEN.Easing.Quadratic.Out)
            .onUpdate(() => {
              _blockPositions.value[b].x = obj.x;
              _blockPositions.value[b].y = obj.y;
            })
            .start();
        }
      }
    };

    const _loadImageToCanvas = () => {
      const sourceImgEl = sourceImg.value;
      const canvas = puzzleBoardRef.value.querySelector('.puzzle-canvas');
      const ctx = canvas.getContext('2d');
      const w = props.width;
      const h = props.height;
      const vw = sourceImgEl.width;
      const vh = sourceImgEl.height;
      const ratio = Math.max(w / vw, h / vh);

      _tmpCanvas.width = vw * ratio;
      _tmpCanvas.height = vh * ratio;
      _tmpCtx.drawImage(sourceImgEl, 0, 0, vw * ratio, vh * ratio);

      const marginX = (vw * ratio - w) / 2;
      const marginY = (vh * ratio - h) / 2;
      ctx.drawImage(_tmpCanvas, marginX, marginY, w, h, w, 0, w, h);
    };

    const _loadVideoFrameToCanvas = () => {
      const sourceImgEl = sourceImg.value;
      const canvas = puzzleBoardRef.value.querySelector('.puzzle-canvas');
      const ctx = canvas.getContext('2d');
      const w = internalWidth.value;
      const h = internalHeight.value;
      const vw = sourceImgEl.videoWidth;
      const vh = sourceImgEl.videoHeight;
      const ratio = Math.max(w / vw, h / vh);

      _tmpCanvas.width = vw * ratio;
      _tmpCanvas.height = vh * ratio;
      _tmpCtx.drawImage(sourceImgEl, 0, 0, vw * ratio, vh * ratio);

      const marginX = (vw * ratio - w) / 2;
      const marginY = (vh * ratio - h) / 2;
      ctx.drawImage(_tmpCanvas, marginX, marginY, w, h, w, 0, w, h);
    };

    const slide = (idx) => {
      if (!_isStarted.value) {
        _isStarted.value = true;
        emit('start');
      }
      board.value.slide(idx);
      blocks.value = [...board.value.blocks];
    };

    return {
      blocks,
      isGoal,
      manhattan,
      hamming,
      internalWidth,
      internalHeight,
      board,
      rafId,
      slide,
      onResize,
      cellWidth,
      cellHeight,
      canvasStyle,
      sourceStyle,
      _tmpCanvas,
      _tmpCtx,
      puzzleBoardRef,
    };
  }
};
</script>

<style scoped>
#sourceImg,
#targetImg {
  width: 300px;
  height: 300px;
}
.puzzle-canvas {
  position: absolute;
  margin: 0;
  padding: 0;
  top: 0;
  left: 0;
  width: 200%;
  height: 100%;
}
.puzzle-message {
  position: absolute;
  width: 100%;
  height: 100%;
}
.puzzle-board {
  position: absolute;
  overflow: hidden;
  width: 100%;
  height: 100%;
}
.tile-number {
  position: absolute;
  text-shadow: 1px 1px 0 #222;
  color: #fafafa;
}
</style>
