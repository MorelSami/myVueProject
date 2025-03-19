<script setup>
  import { defineProps, defineEmits, nextTick, ref } from 'vue';

  const props = defineProps(['todo']); //status filter attribute
  const emit = defineEmits(['delete', 'update']);  //event handler to be used to pass the data to the component
  const editing = ref(false);
  const titleField = ref(null);
  const newTitle = ref(props.todo.title);

  const rename = () => {
    if (!editing.value) {
        return;
    }

    if (newTitle.value === props.todo.title) {
      return;
    }

    editing.value = false;
    newTitle.value = emit('update', {...props.todo, title: newTitle.value});
  };

  const startEditing = async () => {
    newTitle.value = props.todo.title;
    editing.value = true;

    await nextTick();
  
    if (titleField.value) {
      titleField.value.focus();
    }
  };

</script>

<template>
  <div data-cy="Todo" class="todo" :class="{completed : todo.completed}">
    <label class="todo__status-label">
      <input
        data-cy="TodoStatus"
        type="checkbox"
        class="todo__status"
        :checked="todo.completed"
        @change="emit('update', { ...todo, completed: !todo.completed })"
      />
    </label>
    <form v-if="editing" @submit.prevent="rename">
      <input
        ref="titleField"
        v-model="newTitle"
        @keyup.escape="editing = false"
        @blur="rename"
        type="text"
        class="todo__title-field"
        placeholder="Empty todo will be deleted"
      />
    </form>
    <template v-else>
      <span data-cy="TodoTitle" class="todo__title" @dblclick="startEditing">
        {{ todo.title }}
      </span>
      <!-- Remove button appears only on hover -->
      <button type="button" class="todo__remove" data-cy="TodoDelete" @click="emit('delete')">
        ×
      </button>
    </template>
    <!-- overlay will cover the todo while it is being deleted or updated -->
    <div data-cy="TodoLoader" class="modal overlay" :class="{'is-active' : false}">
      <div class="modal-background has-background-white-ter"></div>
      <div class="loader"></div>
    </div>
  </div>
</template>

<style scoped>
</style>
