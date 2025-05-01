<template>
    <div class="border rounded-md p-4 shadow bg-white">
      <div class="flex justify-between items-center">
        <div>
          <h2 class="font-bold text-lg text-blue-700">
            {{ person.name }}
            <span class="text-sm text-gray-400 ml-2">(descendants: {{ descendantCount }})</span>
          </h2>
          <p class="text-sm text-gray-600">{{ person.position }}</p>
          <p class="text-sm text-gray-500">💰 ${{ person.salary.toLocaleString() }}</p>
          <p class="text-sm text-gray-500">🧑‍💼 IC Cost: ${{ icCost.toLocaleString(undefined, { maximumFractionDigits: 0 }) }}</p>
          <p class="text-sm text-gray-500">👨‍💼 Manager Cost: ${{ managementCost.toLocaleString(undefined, { maximumFractionDigits: 0 }) }}</p>
          <p class="text-sm text-gray-500">💼 Total Cost: ${{ totalCost.toLocaleString(undefined, { maximumFractionDigits: 0 }) }}</p>
        </div>
  
        <button
          v-if="person.children?.length"
          @click="expanded = !expanded"
          class="text-sm px-2 py-1 bg-blue-100 text-blue-700 rounded hover:bg-blue-200"
        >
          {{ expanded ? 'Collapse' : 'Expand' }}
        </button>
      </div>
  
      <!-- Recursively render children with depth limit -->
      <div
        v-if="expanded && person.children?.length && depth < maxDepth"
        class="ml-6 mt-4 border-l pl-4 space-y-3"
      >
        <OrgNode
          v-for="child in person.children"
          :key="child.id"
          :person="child"
          :depth="depth + 1"
        />
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed } from 'vue'
  import OrgNode from './OrgNode.vue'
  
  const props = defineProps({
    person: Object,
    depth: {
      type: Number,
      default: 0
    }
  })
  
  const maxDepth = 3
  const expanded = ref(true)
  
  const descendantCount = computed(() => {
    const countChildren = (node) => {
      if (!node.children) return 0
      return node.children.length + node.children.reduce((sum, child) => sum + countChildren(child), 0)
    }
    return countChildren(props.person)
  })
  
  const icCost = computed(() => {
    const sumIC = (node) => {
      if (!node.children || node.children.length === 0) {
        return node.salary
      }
      return node.children.reduce((sum, child) => sum + sumIC(child), 0)
    }
    return sumIC(props.person)
  })
  
  const managementCost = computed(() => {
    const sumManagers = (node) => {
      if (!node.children || node.children.length === 0) return 0
      const selfCost = node.salary
      const childrenCost = node.children.reduce((sum, child) => sum + sumManagers(child), 0)
      return selfCost + childrenCost
    }
    return sumManagers(props.person)
  })
  
  const totalCost = computed(() => {
    return icCost.value + managementCost.value
  })
  </script>
  
  