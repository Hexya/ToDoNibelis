<script setup>
import { ref } from "vue";
import TaskBoard from "./TaskBoard.vue";

// ADD A UNIQUE ID
function generateId() {
  return Date.now().toString(36) + Math.random().toString(36).substr(2, 5);
}

const newTask = ref("");
const tasks = ref([]);

function addTask() {
  if (newTask.value.trim() !== "") {
    tasks.value.push({
      id: generateId(),
      text: newTask.value.trim(),
      done: false,
    });
    newTask.value = "";
  }
}

function toggleTask(index) {
  tasks.value[index].done = !tasks.value[index].done;
}

function deleteTask(index) {
  tasks.value.splice(index, 1);
}
</script>

<template>
  <div class="todo-list-container">
    <TaskBoard :tasks="tasks" @toggle="toggleTask" @delete="deleteTask">
      <form @submit.prevent="addTask" class="form">
        <input v-model="newTask" type="text" placeholder="New task" />
        <button type="submit">
          <img src="./Images/send.png" alt="send" />
        </button>
      </form>
    </TaskBoard>
  </div>
</template>

<style scoped lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Reenie+Beanie&display=swap");

.todo-list-container {
  display: flex;
  justify-content: center;
  width: 100%;
  align-items: center;
  min-height: 100vh;
  background: hsl(220, 7%, 24%);
  padding: 2rem;
  font-family: "Reenie Beanie", cursive;
  position: relative;
}

.form {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  bottom: 10px;
  display: flex;
  gap: 0.5rem;
  width: 100%;
  max-width: 400px;

  input {
    flex: 1;
    padding: 0.5rem;
    background: #222;
    border: 1px solid #555;
    color: white;
    border-radius: 4px;
    outline: none;
  }

  button {
    background: white;
    color: #000;
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 4px;
    font-weight: bold;
    cursor: pointer;

    &:hover {
      background: #ddd;
    }

    img {
      height: 10px;
      width: 10px;
    }
  }
}
</style>
