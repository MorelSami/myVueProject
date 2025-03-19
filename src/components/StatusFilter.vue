<script setup>
  import { defineProps, defineEmits } from 'vue';

  const { modelValue } = defineProps(['modelValue']); //status filter attribute
  const emit = defineEmits(['update:modelValue']);  //event handler to be used to pass the data to the component
</script>

<template>
   <!-- Active link should have the 'selected' class -->
   <nav class="filter" data-cy="Filter">
      <a
        href="#/"
        class="filter__link"
        :class="{selected : modelValue === 'all'}"
        data-cy="FilterLinkAll"
        @click = "emit('update:modelValue', 'all')"
      >
        All
      </a>
      <a
        href="#/active"
        class="filter__link"
        :class="{selected : modelValue === 'active'}"
        data-cy="FilterLinkActive"
       @click = "emit('update:modelValue', 'active')"
      >
        Active
      </a>
      <a
        href="#/completed"
        class="filter__link"
        :class="{selected : modelValue === 'completed'}"
        data-cy="FilterLinkCompleted"
        @click = "emit('update:modelValue', 'completed')"
      >
        Completed
      </a>
    </nav>
</template>
<script scoped>
</script>