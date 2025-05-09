<script setup>
// Vue core imports
import { ref, computed } from 'vue'
// CSV parsing library
import Papa from 'papaparse'
// Reusable org chart node component
import OrgNode from './components/OrgNode.vue'

// Reactive state for employee list and drag event
const people = ref([])
const isDragging = ref(false)

// Handles manual CSV file upload
function handleFileUpload(e) {
  const file = e.target.files[0]
  Papa.parse(file, {
    header: true,
    skipEmptyLines: true,
    complete: (results) => {
      // Parse CSV rows into a simplified data structure
      people.value = results.data.map((entry) => ({
        id: entry["Employee Id"],
        name: entry["Name"],
        position: entry["Job Title"],
        managerId: entry["Manager"] || null,
        salary: parseFloat(entry["Salary"]?.replace(/[^0-9.-]+/g, "") || "0"),
      }))
      buildHierarchy()
    }
  })
}

// Handles drag-and-drop CSV upload
function handleDrop(e) {
  isDragging.value = false
  if (e.dataTransfer?.files?.length) {
    handleFileUpload({ target: { files: e.dataTransfer.files } })
  }
}

// Builds hierarchical tree by assigning children to managers
function buildHierarchy() {
  const map = Object.fromEntries(people.value.map(p => [p.id, p]))
  people.value.forEach(p => {
    if (p.managerId !== null) {
      const manager = map[p.managerId]
      if (manager) {
        manager.children = manager.children || []
        manager.children.push(p)
      }
    }
  })
}

// Get the root node (e.g. CEO with no manager)
const rootPerson = computed(() => people.value.find(p => p.managerId === null))
</script>

<template>
  <div class="min-h-screen p-6 max-w-3xl mx-auto space-y-6">
    <!-- Header -->
    <h1 class="text-3xl font-bold text-center text-purple-700 dark:text-purple-300">
      👥 Agentnoon Org Chart Visualizer
    </h1>
    <p class="text-center text-gray-600 dark:text-gray-400 max-w-xl mx-auto">
      Upload a CSV file to visualize your company hierarchy. Calculates IC & manager cost, total salaries, and ratio insights. Built to scale with 40,000+ employees.
    </p>

    <!-- File Upload UI -->
    <div
      class="w-full max-w-xl mx-auto p-6 bg-white dark:bg-gray-800 rounded-lg shadow-md text-center border-2 border-dashed"
      :class="isDragging ? 'border-blue-500' : 'border-gray-300'"
      @dragover.prevent="isDragging = true"
      @dragleave.prevent="isDragging = false"
      @drop.prevent="handleDrop"
    >
      <p class="text-gray-700 dark:text-gray-200 mb-4">
        Drag and drop your CSV file here, or
      </p>
      <input
        type="file"
        accept=".csv"
        @change="handleFileUpload"
        class="hidden"
        ref="fileInput"
      />
      <button
        @click="$refs.fileInput.click()"
        class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
      >
        Browse Files
      </button>
      <p class="mt-3 text-xs text-gray-500 dark:text-gray-400">
        Sample: <code>giga-corp.csv</code> with 40,000+ employees
      </p>
    </div>

    <!-- Parsing confirmation -->
    <div v-if="people.length" class="text-green-700 dark:text-green-400 text-center font-medium">
      ✅ Parsed {{ people.length }} people successfully!
    </div>

    <!-- Org Chart Root Render -->
    <OrgNode v-if="rootPerson" :person="rootPerson" :depth="0" class="mt-6" />
  </div>
</template>

<style scoped></style>
