<script setup lang="ts">
import type { Task } from '../types/task'
import TaskItem from './TaskItem.vue'

defineProps<{
  tasks: Task[]
}>()

const emit = defineEmits<{
  toggle: [id: number]
  remove: [id: number]
}>()
</script>

<template>
  <div v-if="tasks.length" class="task-list">
    <TaskItem
      v-for="task in tasks"
      :key="task.id"
      :task="task"
      @toggle="emit('toggle', $event)"
      @remove="emit('remove', $event)"
    />
  </div>

  <div v-else class="empty">
    <span class="empty-symbol">☽</span>
    <p>Na razie panuje tu cisza...</p>
    <span>Dodaj zadanie i zacznij działać.</span>
  </div>
</template>