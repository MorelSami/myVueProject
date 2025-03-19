<script setup>
  import { computed, ref, watch, onBeforeMount, onMounted } from 'vue';
  import _todos from './data/todos';
  import * as todoApi from './api/todos';
  import StatusFilter from './components/StatusFilter.vue';
  import TodoItem from './components/TodoItem.vue';

  const todos = ref([]);
  onMounted(async () => {
    todos.value = await todoApi.getTodos();
  });

  const status = ref("all");
  const title = ref("");
  const errorMessage = ref("");
  const activeTodos = computed(() => todos.value.filter(todo => !todo.completed));

  const filteredTodos = computed(() => {
    if (status.value == 'active') {
      return activeTodos.value;
    }

    if (status.value == 'completed') {
      return todos.value.filter(todo => todo.completed);
    }

    return todos.value;
  });

  const addTodo = async () => {
    if (!title.value) {
      errorMessage = "Title should not be empty";
    }

    const newTodo = await todoApi.createTodo(title.value);

    todos.value.push(newTodo);
    title.value = '';
  };
  
  const deleteTodo = async todoId => {
    await todoApi.deleteTodo(todoId);
    todos.value = todos.value.filter(todo => todoId !== todo.id);
  };

  const updateTodo = async ({ id, title, completed }) => {
    const updatedTodo = await todoApi.updateTodo({ id, title, completed });
    const currentTodo = todos.value.find(todo => todo.id === id);
  
    Object.assign(currentTodo, updatedTodo);
  };
</script>

<template>

  <div class="todoapp">
    <h1 class="todoapp__title">todos</h1>
    <div class="todoapp__content">
      <header class="todoapp__header">
        <!--this button should have `active` class only if all todos are completed -->
        <button
          :class="{active : activeTodos.length === 0}"
          type="button"
          class="todoapp__toggle-all active"
          data-cy="ToggleAllButton"
        ></button>
        <!-- Add a todo on form submit -->
        <form @submit.prevent="addTodo">
          <input
            data-cy="NewTodoField"
            type="text"
            class="todoapp__new-todo"
            placeholder="What needs to be done?"
            v-model="title"
            @input=""
          />
        </form>
      </header>
      <TransitionGroup tag ="section" name="todolist" class="todoapp__main" data-cy="TodoList" v-if="todos.length > 0">
        <!--This is a completed todo -->
        <TodoItem 
          v-for="(todo, i) of filteredTodos" 
          :key="todo.id" 
          :todo="todo" 
          @delete="deleteTodo(todo.id)" 
          @update="updateTodo($event)"
        />
      </TransitionGroup>
      <!-- Hide the footer if there are no todos -->
      <footer class="todoapp__footer" data-cy="Footer" v-if="todos.length > 0">
        <span class="todo-count" data-cy="TodosCounter">
          {{ activeTodos.length }} items left
        </span>
       <StatusFilter v-model="status" />
        <!-- this button should be disabled if there are no completed todos -->
        <button
          type="button"
          class="todoapp__clear-completed"
          data-cy="ClearCompletedButton"
          :disabled="todos.length === activeTodos.length"
        >
          Clear completed
        </button>
      </footer>
    </div>
    <!-- DON'T use conditional rendering to hide the notification -->
    <!-- Add the 'hidden' class to hide the message smoothly -->
    <div
      data-cy="ErrorNotification"
      class="notification is-danger is-light has-text-weight-normal"
      :class = "{hidden: !errorMessage}"
    >
      <button data-cy="HideErrorButton" type="button" class="delete" @click="errorMessage=''"></button>
      <!-- show only one message at a time -->
      {{ errorMessage }}
      <br />
    </div>
  </div>
</template>

<style scoped>

.todolist-enter-active, .todolist-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}

.todolist-enter-from, .todolist-leave-to {
  opacity: 0;
  max-height: 0;
  transform: scaleY(0);
}

header {
  line-height: 1.5;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
