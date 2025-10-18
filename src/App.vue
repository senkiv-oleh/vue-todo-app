<script setup>
  import { ref, computed } from 'vue';
  import originalTools from './data/todo.js'

  const todos = ref(originalTools);
  const title = ref("")
  const errorMessage = ref("");
  const status = ref('all')

  const activeTodos = computed(() =>
      todos.value.filter((todo) => !todo.completed)
  );

  function addTodo() {
    if(!title.value) {
      errorMessage.value = "Title should not be empty";

      return;
    }

    todos.value.push({
      id: Date.now(),
      title: title.value,
      completed: false,
    });

    title.value = "";
  }

  const visibleTodos = computed(() => {
    if (status.value === 'active') {
      return activeTodos.value
    }

    if (status.value === 'completed') {
      return todos.value.filter((todo) => todo.completed)
    }

    return todos.value;
  })

</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos {{ todos.length}}</h1>

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

      <section class="todoapp__main">
        <div class="todo" v-for="(todo, i) of visibleTodos" :key="todo.id" :class="{completed: todo.completed}">
          <label class="todo__status-label">
            <input
                type="checkbox"
                class="todo__status"
                v-model="todo.completed"
            />
          </label>

          <form v-if="false">
            <input
            type="text"
            class="todo__title-field"
            placeholder="Empty todo will be deleted"/>
          </form>

          <template v-else="false">
            <span class="todo__title"> {{todo.title}}</span>
            <button class="todo__remove" @click="todos.splice(i, 1)">x</button>
          </template>

          <div class="modal overlay" :class="{ 'is-active': false }">
            <div class="modal-background has-background-white-ter" />
            <div class="loader"></div>
          </div>
        </div>
      </section>

      <!-- Hide the footer if there are no todos -->
      <footer class="todoapp__footer">
          <span class="todo-count">
            {{ activeTodos.length }} items left
          </span>

        <!-- Active link should have the 'selected' class -->
        <nav class="filter">
          <a
            href="#/"
            class="filter__link"
            :class="{ selected: status === 'all'}"
            @click="status = 'all'"
          >
            All
          </a>

          <a
              href="#/active"
              class="filter__link"
              :class="{ selected: status === 'active' }"
              @click="status = 'active'"
          >
            Active
          </a>

          <a
              href="#/completed"
              class="filter__link"
              :calss="{ selected: atatus === 'completed' }"
              @click=" status = 'completed' "
          >
            Completed
          </a>
        </nav>

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
      <button type="button" class="delete" @click="errorMessage = ''" ></button>
      {{errorMessage}}

    </div>
  </div>
 </template>


<style scoped></style>
