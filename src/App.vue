
<script setup lang="ts">
import { computed, ref } from 'vue'
import TaskList from './components/TaskList.vue'
import type { Task, Priority } from './types/task'

type Filter = 'all' | 'active' | 'done'

const tasks = ref<Task[]>([
  {
    id: 1,
    title: 'Dokończyć projekt aplikacji',
    completed: false,
    priority: 'high',
    dueDate: '',
  },
  {
    id: 2,
    title: 'Przejrzeć notatki z lekcji',
    completed: true,
    priority: 'low',
    dueDate: '',
  },
  {
    id: 3,
    title: 'Przygotować się na jutro',
    completed: false,
    priority: 'medium',
    dueDate: '',
  },
])

const newTask = ref('')
const newPriority = ref<Priority>('medium')
const newDueDate = ref('')
const filter = ref<Filter>('all')
const error = ref('')

const total = computed(() => tasks.value.length)

const doneCount = computed(
  () => tasks.value.filter(task => task.completed).length,
)

const leftCount = computed(() => total.value - doneCount.value)

const highPriorityCount = computed(
  () => tasks.value.filter(
    task => task.priority === 'high' && !task.completed,
  ).length,
)

const filteredTasks = computed(() => {
  if (filter.value === 'active') {
    return tasks.value.filter(task => !task.completed)
  }

  if (filter.value === 'done') {
    return tasks.value.filter(task => task.completed)
  }

  return tasks.value
})

const progress = computed(() => {
  if (!total.value) return 0
  return Math.round((doneCount.value / total.value) * 100)
})

function addTask() {
  const title = newTask.value.trim()

  if (!title) {
    error.value = 'Najpierw wpisz nazwę zadania.'
    return
  }

  tasks.value.unshift({
    id: Date.now(),
    title,
    completed: false,
    priority: newPriority.value,
    dueDate: newDueDate.value,
  })

  newTask.value = ''
  newPriority.value = 'medium'
  newDueDate.value = ''
  error.value = ''
  filter.value = 'all'
}

function toggleTask(id: number) {
  const task = tasks.value.find(item => item.id === id)

  if (task) {
    task.completed = !task.completed
  }
}

function removeTask(id: number) {
  tasks.value = tasks.value.filter(task => task.id !== id)
}

function clearCompleted() {
  tasks.value = tasks.value.filter(task => !task.completed)
}
</script>

<template>
  <main class="app">
    <div class="shell">
      <header class="top">
        <a class="brand" href="#" aria-label="Nocturne — strona główna">
          <span class="logo">☽</span>
          <span>nocturne<span class="brand-dot">.</span></span>
        </a>

        <span class="top-label">YOUR PERSONAL TASK GRIMOIRE</span>
      </header>

      <section class="hero">
        <div class="hero-ornament">✦</div>
        <div class="eyebrow">✧ TWOJE MROCZNE KRÓLESTWO ✧</div>

        <h1>Chaos ma swój <em>porządek.</em></h1>

        <p>
          Nawet najdłuższa noc składa się z małych kroków.
          Zapisz swoje plany i odzyskaj kontrolę.
        </p>

        <div class="stats">
          <div class="stat">
            <span class="stat-number">{{ total }}</span>
            <span class="stat-label">Wszystkich</span>
          </div>

          <div class="stat">
            <span class="stat-number">{{ leftCount }}</span>
            <span class="stat-label">Pozostało</span>
          </div>

          <div class="stat">
            <span class="stat-number">{{ doneCount }}</span>
            <span class="stat-label">Ukończonych</span>
          </div>
        </div>
      </section>

      <section class="progress-section">
        <div class="progress-heading">
          <span>POSTĘP RYTUAŁU</span>
          <strong>{{ progress }}%</strong>
        </div>

        <div
          class="progress-track"
          role="progressbar"
          :aria-valuenow="progress"
          aria-valuemin="0"
          aria-valuemax="100"
          aria-label="Postęp wykonania zadań"
        >
          <div class="progress-fill" :style="{ width: `${progress}%` }"></div>
        </div>
      </section>

      <section class="panel">
        <div class="panel-head">
          <div>
            <div class="section-kicker">ROZDZIAŁ I</div>
            <h2>Lista zadań</h2>
          </div>

          <span class="counter">{{ leftCount }} aktywnych</span>
        </div>

        <form class="add-form" @submit.prevent="addTask">
          <label class="field-label" for="task-title">Nowe zadanie</label>

          <input
            id="task-title"
            v-model="newTask"
            maxlength="120"
            placeholder="Co dziś trzeba zrobić?"
            autocomplete="off"
            @input="error = ''"
          />

          <div class="form-row">
            <div class="field-group">
              <label class="field-label" for="task-priority">Priorytet</label>
              <select id="task-priority" v-model="newPriority">
                <option value="low">☽ Niski</option>
                <option value="medium">✧ Średni</option>
                <option value="high">✦ Wysoki</option>
              </select>
            </div>

            <div class="field-group">
              <label class="field-label" for="task-date">Termin wykonania</label>
              <input id="task-date" v-model="newDueDate" type="date" />
            </div>
          </div>

          <p v-if="error" class="error" role="alert">{{ error }}</p>

          <button class="add-btn" type="submit">
            <span>＋</span> Dodaj do grymuaru
          </button>
        </form>

        <div class="list-heading">
          <h3>Twoje plany</h3>
          <span class="subtle">{{ total }} wpisów</span>
        </div>

        <div class="filters" aria-label="Filtruj zadania">
          <button
            class="filter"
            :class="{ active: filter === 'all' }"
            :aria-pressed="filter === 'all'"
            @click="filter = 'all'"
          >
            Wszystkie
          </button>

          <button
            class="filter"
            :class="{ active: filter === 'active' }"
            :aria-pressed="filter === 'active'"
            @click="filter = 'active'"
          >
            Aktywne
          </button>

          <button
            class="filter"
            :class="{ active: filter === 'done' }"
            :aria-pressed="filter === 'done'"
            @click="filter = 'done'"
          >
            Ukończone
          </button>
        </div>

        <TaskList
          :tasks="filteredTasks"
          @toggle="toggleTask"
          @remove="removeTask"
        />

        <footer class="panel-foot">
          <span v-if="highPriorityCount">
            <span class="warning-symbol">✦</span>
            {{ highPriorityCount }} zadań o wysokim priorytecie
          </span>

          <span v-else>✧ Wszystko pod kontrolą</span>

          <button class="clear-btn" type="button" @click="clearCompleted">
            Usuń ukończone
          </button>
        </footer>
      </section>

      <footer class="page-footer">
        <span>NOCTURNE</span>
        <span>PER ASPERA AD ASTRA · ✦</span>
      </footer>
    </div>
  </main>
</template>
