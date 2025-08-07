<template>
  <div class="min-h-screen bg-gradient-to-br from-purple-600 to-blue-500 p-8 flex items-center justify-center">
    <div class="max-w-2xl w-full bg-white rounded-2xl shadow-2xl p-8">
      <h1 class="text-4xl font-extrabold text-gray-800 mb-8 text-center">Todo list</h1>

      <div class="mb-4 text-center text-lg">
        <p class="font-semibold text-gray-600">Broj zadataka: {{ taskCount }}</p>
      </div>

      <div class="flex mb-6 justify-center">
        <input 
          v-model="newTask" 
          @keyup.enter="addTask" 
          placeholder="Add new task..."
          class="flex-1 p-4 text-lg border border-gray-300 rounded-l-xl focus:outline-none focus:ring-2 focus:ring-purple-500"
        />
        <button
          @click="addTask"
          class="bg-purple-600 text-white px-6 py-4 rounded-r-xl hover:bg-purple-800 transition-all"
        >
          ➕ Add
        </button>
      </div>

      <p v-if="tasks.length === 0" class="text-center text-lg text-gray-500">
        Nema zadataka. Dodaj neki!
      </p>

      <ul v-if="tasks.length > 0" class="space-y-4">
        <li v-for="(task, index) in tasks" :key="task.id"  
            class="flex items-center justify-between p-4 rounded-xl shadow-sm transition-all duration-300 bg-gray-100 hover:bg-gray-200">

          <div 
            @click="toggleCompleted(index)"
            :class="['text-lg cursor-pointer select-none', task.completed ? 'line-through text-gray-400' : 'text-gray-800']">
            {{ task.text }}
          </div>

          <div class="space-x-3">
            <button @click="editTask(index)" class="text-blue-500 hover:text-blue-700 text-xl">
              ✏️
            </button>
            <button @click="deleteTask(index)" class="text-red-500 hover:text-red-700 text-xl">
              ❌
            </button>
          </div>
        </li>
      </ul>
    </div>
  </div>

  <div>
    <FooterComponent />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import FooterComponent from './components/FooterComponent.vue'

import { db } from './firebase/firebase'
import { collection, addDoc, getDocs, deleteDoc, doc, updateDoc, serverTimestamp } from 'firebase/firestore'

// Reactive variables
const newTask = ref('')
const tasks = ref([])

// Computed property
const taskCount = computed(() => tasks.value.length)

// Methods
const addTask = async () => {
  if (newTask.value.trim() !== '') {
    const docRef = await addDoc(collection(db, 'todos'), {
      text: newTask.value,
      createdAt: serverTimestamp(),
      completed: false
    })
    tasks.value.push({ id: docRef.id, text: newTask.value, completed: false })
    newTask.value = ''
  }
}

const editTask = async (index) => {
  const updatedText = prompt('Edit task', tasks.value[index].text)
  if (updatedText !== null) {
    const task = tasks.value[index]
    await updateDoc(doc(db, 'todos', task.id), { text: updatedText })
    task.text = updatedText
  }
}

const deleteTask = async (index) => {
  const task = tasks.value[index]
  await deleteDoc(doc(db, 'todos', task.id))
  tasks.value.splice(index, 1)
}

const toggleCompleted = async (index) => {
  const task = tasks.value[index]
  const newStatus = !task.completed
  await updateDoc(doc(db, 'todos', task.id), { completed: newStatus })
  task.completed = newStatus
}

// Lifecycle hook
onMounted(async () => {
  alert('Mounted radi nešto čim se stvar pokrene...sad nam dohvaća podatke iz kolekcije u firebaseu (jako cool stvar kad skužite)')
  const querySnapshot = await getDocs(collection(db, 'todos'))
  querySnapshot.forEach((docSnap) => {
    tasks.value.push({ id: docSnap.id, ...docSnap.data() })
  })
})
</script>

<style>
body {
  margin: 0;
  font-family: 'Inter', sans-serif;
  background: linear-gradient(135deg, #1f2937, #4b5563, #6b7280, #374151);
  background-size: 400% 400%;
  animation: gradientBG 10s ease infinite;
}

@keyframes gradientBG {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.max-w-2xl {
  width: 100%;
}

.flex {
  display: flex;
}

.items-center {
  align-items: center;
}

.justify-center {
  justify-content: center;
}

.text-center {
  text-align: center;
}

.space-x-3 {
  margin-right: 1rem;
}
</style>
