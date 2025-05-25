<script setup>
import { ref, computed, onMounted, nextTick, watch } from "vue";

const props = defineProps({
  tasks: Array,
});
const emit = defineEmits(["toggle", "delete"]);

const board = ref(null);
const rotateX = ref(0);
const rotateY = ref(0);
const glareX = ref("50%");
const glareY = ref("50%");
const isHovering = ref(false);
const positions = ref({});

// GENERATE POSITION ON BOARD
function randomPosition() {
  if (!board.value) return { top: "0px", left: "0px", rotation: "0deg" };
  const rect = board.value.getBoundingClientRect();
  const maxTop = rect.height - 250;
  const maxLeft = rect.width - 250;
  
  return {
    top: Math.random() * maxTop + "px",
    left: Math.random() * maxLeft + "px",
    rotation: (Math.random() * 100) - 50 + "deg",
  };
}

onMounted(async () => {
  await nextTick();
  const pos = {};
  props.tasks.forEach((task) => {
    pos[task.id] = randomPosition();
  });
  positions.value = pos;
});

// ON TASK ADD OR DELETE
watch(
  () => props.tasks.length,
  (newLength, oldLength) => {
    if (newLength > oldLength) {
      // ADD POS FOR NEW ID
      const newTask = props.tasks[props.tasks.length - 1];
      positions.value[newTask.id] = randomPosition();
    } else if (newLength < oldLength) {
      // DELETE POS FROM ID DELETE
      const taskIds = props.tasks.map((t) => t.id);
      Object.keys(positions.value).forEach((id) => {
        if (!taskIds.includes(id)) {
          delete positions.value[id];
        }
      });
    }
  }
);

// UPDATE POS AND GLARE
function handleMouseMove(e) {
  const el = board.value;
  if (!el) return;

  const rect = el.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;

  const centerX = rect.width / 2;
  const centerY = rect.height / 2;

  rotateX.value = ((y - centerY) / centerY) * -5;
  rotateY.value = ((x - centerX) / centerX) * 5;

  glareX.value = `${x}px`;
  glareY.value = `${y}px`;

  isHovering.value = true;

}

// RESET POS ON MOUSELEAVE
function resetRotation() {
  const el = board.value;
  if (!el) return;

  rotateX.value = 0;
  rotateY.value = 0;
  glareX.value = "50%";
  glareY.value = "50%";

  isHovering.value = false;
}

// CHANGE COLOR OF GLARE
const glareColors = ['rgba(255, 255, 255, 0.3)', 'blue', 'cyan', 'fuchsia'];
const currentGlareIndex = ref(0);

const currentGlareColor = computed(() => glareColors[currentGlareIndex.value]);

function cycleGlareColor() {
  currentGlareIndex.value = (currentGlareIndex.value + 1) % glareColors.length;
}
</script>

<template>
  <div class="task-board-container">
      <div
      class="board"
      :class="{ 'is-resetting': !isHovering}"
      ref="board"
      :style="{
            transform: `rotateX(${rotateX}deg) rotateY(${rotateY}deg)`,
            '--glare-x': glareX,
            '--glare-y': glareY,
            '--glare-color': currentGlareColor,
        }"
      @mousemove="handleMouseMove"
      @mouseleave="resetRotation"
      >
      <h1>Todo board</h1>
      <img class="logo" src="./Logo.png" alt="logo" @click="cycleGlareColor"/>
      <div class="task-list">
        <p
          v-for="task in tasks"
          :key="task.id"
          :style="{
            position: 'absolute',
            top: positions[task.id]?.top || '0px',
            left: positions[task.id]?.left || '0px',
            maxWidth: '250px',
          }"
        >
          <span
            class="task-text"
            :class="{ done: task.done }"
            @click="$emit('toggle', tasks.indexOf(task))"
          >
            {{ task.text }}
          </span>
          <div
            v-if="task.done"
            class="brush"
            :style="{ transform: `rotate(${positions[task.id]?.rotation || '0deg'}) translate(-50%, -50%)` }"
            >
            <div>
                <img src="./brush.png" alt="brush" />
            </div>
        </div>
          <button
            class="delete-btn"
            @click="$emit('delete', tasks.indexOf(task))">
          </button>
        </p>
      </div>
      <slot></slot>
    </div>
  </div>
</template>

<style scoped lang="scss">
.task-board-container {
  perspective: 1000px;
  display: flex;
  justify-content: center;
  width: 90%;
  height: 90vh;

  .board {
    position: relative;
    transform-style: preserve-3d;
    transition: transform 0.1s ease;
    will-change: transform;
    background-color: #111;
    border: 5px solid #444;
    padding: 2rem;
    border-radius: 1rem;
    color: #fff;
    width: 100%;
    box-shadow: 0 3px 1px rgba(0, 0, 0, 0.384), 0 0 3px 1px black,
      0 0 12px 2px #808080, 0px 10px 20px -5px black, 0 0 40px -30px #808080,
      0 0 50px -20px #808080;
  
    &.is-resetting {
      transition: transform 1s ease;
    }
  
    &::before {
      content: "";
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
      background: radial-gradient(
        circle at var(--glare-x, 50%) var(--glare-y, 50%),
        var(--glare-color),
        transparent 60%
      );
      border-radius: inherit;
      z-index: 1;
      mix-blend-mode: soft-light;
      opacity: 0;
      transition: opacity 0.2s;
    }
  
    &:hover::before {
      opacity: 1;
    }

    .logo {
        position: absolute;
        top: 20px;
        left: calc(50% - 50px);
        width: 100px;
        cursor: pointer;
    }
  
    h1 {
        margin-top: 30px;
      text-align: center;
      font-size: 1rem;
      margin-bottom: 1rem;
      color: #fff;
    }
  }
  
  .task-list {
    list-style: none;
    padding: 0;
    position: relative; 
  
    p {
      font-size: 1.5rem;
      line-height: 1.5rem;
      max-width: 250px;
  
      &:hover {
        .delete-btn {
          opacity: 1;
        }
      }
  
      .task-text {
        cursor: pointer;
        position: relative;
        display: inline-block;
        user-select: none;
        transition: opacity 0.25s;
        &.done {
          opacity: 0.5;
        }
      }
  
      .brush {
          height: 30px;
          width: 150px;
          position: absolute;
          top: 50%;
          left: 50%;
          overflow: hidden;
          pointer-events: none;
          transform-origin: left;
          
          div {
              height: 30px;
              width: 0px;
              overflow: hidden;
              animation: brushed 0.15s ease-in-out forwards;
              opacity: 0.5;
  
              img {
                  height: 30px;
                  width: 150px;
              }
  
              @keyframes brushed {
                  0% {
                      width: 0;
                  }
                  100% {
                      width: 150px;
                  }
              }
          }
      }
  
      .delete-btn {
        position: absolute;
        top: calc(50% - 7px);
        right: -25px;
        background: transparent;
        background-image: url("./cross.png");
        background-size: cover;
        height: 15px;
        width: 15px;
        filter: invert(1);
        border: none;
        color: white;
        font-size: 1.2rem;
        cursor: pointer;
        transition: transform 0.2s, opacity 0.25s;
        opacity: 0;
  
        &:hover {
          transform: scale(1.2);
        }
      }
    }
  }
}
</style>
