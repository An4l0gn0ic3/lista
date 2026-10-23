
<script setup lang="ts">
import type { Task, Priority } from '../types/task'

defineProps<{
  task: Task
}>()

const emit = defineEmits<{
  toggle: [id: number]
  remove: [id: number]
}>()

const priorityLabels: Record<Priority, string> = {
  low: 'Niski',
  medium: 'Średni',
  high: 'Wysoki',
}

function formatDate(date: string): string {
  if (!date) return 'Bez terminu'

  const [year, month, day] = date.split('-')
  return `${day}.${month}.${year}`
}

function isOverdue(task: Task): boolean {
  if (!task.dueDate || task.completed) return false

  const today = new Date()
  today.setHours(0, 0, 0, 0)

  return new Date(`${task.dueDate}T00:00:00`) < today
}
</script>

<template>
  <article class="task" :class="[
    { done: task.completed, overdue: isOverdue(task) },
    `priority-${task.priority}`,
  ]">
    <button
      class="check"
      :class="{ checked: task.completed }"
      :aria-label="task.completed ? 'Oznacz jako niewykonane' : 'Oznacz jako wykonane'"
      @click="emit('toggle', task.id)"
    >
      <svg v-if="task.completed" viewBox="0 0 20 20" fill="none">
        <path
          d="m4 10 4 4 8-8"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
      </svg>
    </button>

    <div class="task-content">
      <span class="task-name">{{ task.title }}</span>

      <div class="task-meta">
        <span class="priority-badge">
          <span class="priority-dot"></span>
          {{ priorityLabels[task.priority] }} priorytet
        </span>

        <span class="due-date" :class="{ 'date-overdue': isOverdue(task) }">
          <span aria-hidden="true">◷</span>
          {{ formatDate(task.dueDate) }}
          <span v-if="isOverdue(task)" class="overdue-label">Po terminie</span>
        </span>
      </div>
    </div>

    <button
      class="delete"
      aria-label="Usuń zadanie"
      title="Usuń zadanie"
      @click="emit('remove', task.id)"
    >
      ×
    </button>
  </article>
</template>
