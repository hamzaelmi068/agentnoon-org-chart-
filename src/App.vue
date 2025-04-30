<template>
    <div class="min-h-screen p-6 max-w-3xl mx-auto space-y-6">
      <h1 class="text-3xl font-bold text-center text-blue-600">📊 Upload Org Data</h1>
  
      <!-- File input -->
      <input
        type="file"
        accept=".csv"
        @change="handleFileUpload"
        class="block w-full text-sm text-gray-700 border border-gray-300 rounded-md p-2"
      />
  
      <!-- Debug: Show uploaded people count -->
      <div v-if="people.length" class="text-green-700">
        ✅ Parsed {{ people.length }} people
      </div>
    </div>
  </template>
  
  
  <script setup>
  import { ref } from 'vue'
  
  const people = ref([])
  
  function handleFileUpload(e) {
    const file = e.target.files[0]
    const reader = new FileReader()
  
    reader.onload = (event) => {
      const lines = event.target.result.split('\n').filter(line => line.trim())
      const headers = lines.shift().split(',').map(h => h.trim())
  
      people.value = lines.map((line, index) => {
        const values = line.split(',')
        const entry = Object.fromEntries(headers.map((h, i) => [h, values[i]?.trim()]))
        entry.id = index
        entry.salary = parseFloat(entry.salary || 0)
        entry.managerId = entry.managerId ? parseInt(entry.managerId) : null
        return entry
      })
    }
  
    reader.readAsText(file)
  }
  </script>
  