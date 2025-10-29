<script setup>
import { ref, onMounted, computed } from "vue";
import StatusFilter from "@/components/StatusFilter.vue";
import TodoItem from "@/components/TodoItem.vue";
import * as todoApi from '@/api/todos';

const todos = ref([]);
const title = ref("");
const errorMessage = ref("");
const status = ref("all");


onMounted(async () => {
  todos.value = await todoApi.getTodos();
});

const activeTodos = computed(() =>
    todos.value.filter((todo) => !todo.completed),
);

const addTodo = async () => {
  if (!title.value) {
    errorMessage.value = "Title should not be empty";

    return;
  }

  const newTodo = await createTodo(title.value);

  todos.value.push(newTodo);
  title.value = "";
}

const deleteTodo = async todoId => {
  await todoApi.deleteTodo(todoId);
  todos.value = todos.value.filter(todo => todoId !== todo.id);
};

const updateTodo = async ({id, title, completed}) => {
  const updatedTodo = await todoApi.updateTodo({id, title, completed});
  const currentTodo = todos.value.find(todo => todo.id === id);

  Object.assign(currentTodo, updatedTodo);
};


const visibleTodos = computed(() => {
  if (status.value === "active") {
    return activeTodos.value;
  }

  if (status.value === "completed") {
    return todos.value.filter((todo) => todo.completed);
  }

  return todos.value;
});
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos {{ todos.length }}</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <button
            v-if="todos.length > 0"
            type="button"
            class="todoapp__toggle-all active"
            :class="{ active: activeTodos.length === 0 }"
        ></button>

        <form @submit.prevent="addTodo">
          <input
              type="text"
              class="todoapp__new-todo"
              placeholder="What needs to be done?"
              v-model="title"
              @input="errorMessage = ''"
          />
        </form>
      </header>

      <TransitionGroup
          tag="section"
          name="todolist"
          class="todoapp__main"
          v-if="todos.length > 0"
      >
        <TodoItem
            v-for="todo in visibleTodos"
            :key="todo.id"
            :todo="todo"
            @delete="deleteTodo(todo.id)"
            @update="updateTodo($event)"
        />
      </TransitionGroup>

      <!-- Hide the footer if there are no todos -->
      <footer class="todoapp__footer">
        <span class="todo-count"> {{ activeTodos.length }} items left </span>

        <!-- Active link should have the 'selected' class -->
        <StatusFilter v-model="status"/>
        <!-- this button should be disabled if there are no completed todos -->
        <button
            type="button"
            class="todoapp__clear-completed"
            :disabled="todos.length === activeTodos.length"
            @click="todos = activeTodos"
        >
          Clear completed
        </button>
      </footer>
    </div>

    <!-- DON'T use conditional rendering to hide the notification -->
    <!-- Add the 'hidden' class to hide the message smoothly -->
    <div
        v-if="errorMessage"
        class="notification is-danger is-light has-text-weight-normal"
    >
      <button type="button" class="delete" @click="errorMessage = ''"></button>
      {{ errorMessage }}
    </div>
  </div>
</template>

<style scoped>
.todolist-enter-active,
.todolist-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}

.todolist-enter-form,
.todolist-leave-to {
  opasity: 0;
  max-height: 0;
  transform: scaleY(0);
}
</style>
