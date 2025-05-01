<script setup>
import { ref, computed } from 'vue'
import Papa from 'papaparse'
import OrgNode from './components/OrgNode.vue'

const people = ref([])

function handleFileUpload(e) {
  const file = e.target.files[0]

  Papa.parse(file, {
    header: true,
    skipEmptyLines: true,
    complete: (results) => {
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

const rootPerson = computed(() => people.value.find(p => p.managerId === null))
</script>

<template>
  <div class="min-h-screen p-6 max-w-3xl mx-auto space-y-6">
    <h1 class="text-3xl font-bold text-center text-blue-600">📊 Upload Org Data</h1>

    <input
      type="file"
      accept=".csv"
      @change="handleFileUpload"
      class="block w-full text-sm text-gray-700 border border-gray-300 rounded-md p-2"
    />

    <div v-if="people.length" class="text-green-700">
      ✅ Parsed {{ people.length }} people
    </div>

    <OrgNode v-if="rootPerson" :person="rootPerson" :depth="0" class="mt-6" />
  </div>
</template>

<style scoped></style>
