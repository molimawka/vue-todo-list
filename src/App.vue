<script setup lang="ts">
import { computed, ref } from 'vue';

const LocalStorageKey = 'todo-list';

type Task = {
  id: number;
  text: string;
  createdTime: number;
  completed: boolean;
  completeTime?: number;
};

const taskList = ref<Array<Task>>(getFromLocalStorage());

const taskCount = computed(() => {
  return taskList.value.length;
});

const completedTaskCount = computed(() => {
  return taskList.value.reduce((prev, acc) => {
    return acc.completed ? ++prev : prev;
  }, 0);
});

const completedTaskPercent = computed(() => {
  if (taskCount.value === 0 || completedTaskCount.value === 0) {
    return 0;
  }
  return ((completedTaskCount.value / taskCount.value) * 100).toFixed(2);
});

const taskInput = ref('');

function addTask() {
  if (!taskInput.value) {
    return;
  }

  taskList.value.unshift({
    id: Date.now(),
    text: taskInput.value,
    createdTime: Date.now(),
    completed: false,
  });

  taskInput.value = '';

  saveToLocalStorage();
}

function onTaskCheckboxChanged(event: Event, task: Task) {
  const checked = (event.target as HTMLInputElement).checked;
  task.completed = checked;
  if (checked) {
    task.completeTime = Date.now();
  }

  saveToLocalStorage();
}

function getFromLocalStorage() {
  const item = localStorage.getItem(LocalStorageKey);
  if (!item) {
    return [];
  }

  try {
    const taskList = JSON.parse(item);
    return taskList;
  } catch (e) {
    localStorage.removeItem(LocalStorageKey);
    return [];
  }
}

function saveToLocalStorage() {
  localStorage.setItem(LocalStorageKey, JSON.stringify(taskList.value));
}
</script>

<template>
  <div class="wrapper">
    <div class="project-info">
      <div class="project-info--title">Vue ToDo List</div>
      <div class="project-info--all">Todo count: {{ taskCount }}</div>
      <div class="project-info--completed-count">Completed count: {{ completedTaskCount }}</div>
      <div class="project-info--completed-percent">Completed percent: {{ completedTaskPercent }}%</div>
    </div>

    <div class="todo-container">
      <div class="todo-form">
        <input
          v-model="taskInput"
          placeholder="Task name"
          type="text"
        />
        <button @click="addTask()">Add task</button>
      </div>

      <div class="todo-list">
        <template v-if="taskList.length > 0">
          <div
            v-for="task of taskList"
            :key="task.id"
            class="task"
            :class="{ 'task--completed': task.completed }"
          >
            <div class="task-name">
              {{ task.text }}
              <input
                :checked="task.completed"
                type="checkbox"
                @change="onTaskCheckboxChanged($event, task)"
              />
            </div>

            <div class="task-info">
              <div class="task-info--created-time">Created: {{ new Date(task.createdTime).toLocaleString('ru') }}</div>
              <div
                v-if="task.completed"
                class="task-info--completed-time"
              >
                Completed: {{ new Date(task.completeTime!).toLocaleString('ru') }}
              </div>
            </div>
          </div>
        </template>
        <template v-else>
          <div class="task-list--empty">No items in list</div>
        </template>
      </div>
    </div>
  </div>
</template>

<style scoped>
.wrapper {
  width: 100%;
  padding: 20px;
  max-width: 1170px;
  min-height: 100vh;
  box-sizing: border-box;
  margin: 0 auto;

  display: flex;
  flex-direction: column;
  flex-wrap: nowrap;

  font-size: 14px;

  gap: 15px;
}

.project-info {
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;

  text-align: center;
}

.project-info--title {
  font-size: 32px;
  font-weight: bold;

  padding-bottom: 10px;
}

.todo-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  gap: 20px;
}

.todo-form {
  display: flex;
  flex-direction: column;
}

.todo-list {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
}

.task {
  display: flex;
  flex-direction: column;
  align-items: center;

  border: 1px solid;
  padding: 10px;
  gap: 5px;
}

.task--completed {
  border-color: rgba(50, 205, 50);
}

.task-name {
  font-size: 16px;
}

.task-info {
  font-size: 12px;
}
</style>
