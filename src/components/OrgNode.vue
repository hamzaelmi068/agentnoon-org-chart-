<template>
    <div :class="['rounded-md p-4 shadow transition-all', cardColor, 'border']">
      <div class="flex justify-between items-start">
        <div class="space-y-1">
          <h2 class="font-bold text-lg text-gray-900">
            {{ person.name }}
            <span class="text-sm text-gray-500 ml-2">(descendants: {{ descendantCount }})</span>
          </h2>
          <p class="text-sm text-gray-700 italic">{{ person.position }}</p>
          <p class="text-sm text-gray-600">💰 {{ format(person.salary) }}</p>
  
          <div class="text-xs text-gray-700 space-y-0.5 mt-2">
            <p>🧑‍💼 IC Cost: {{ format(icCost) }}</p>
            <p>👨‍💼 Manager Cost: {{ format(managementCost) }}</p>
            <p>💼 Total Cost: {{ format(totalCost) }}</p>
            <p>📊 Manager:IC Ratio: {{ ratio }}</p>
          </div>
        </div>
  
        <button
          v-if="person.children?.length"
          @click="expanded = !expanded"
          class="text-xs px-2 py-1 mt-1 bg-white text-gray-700 border rounded hover:bg-gray-100"
        >
          {{ expanded ? 'Collapse' : 'Expand' }}
        </button>
      </div>
  
      <!-- Recursively render children with depth limit -->
      <div
        v-if="expanded && person.children?.length && depth < maxDepth"
        class="ml-6 mt-4 border-l-2 border-gray-300 pl-4 space-y-3"
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
  
  const format = (num) =>
    '$' + Number(num).toLocaleString(undefined, { maximumFractionDigits: 0 })
  
  const descendantCount = computed(() => {
    if (!props.person.children || props.person.children.length === 0) return 0
  
    const countChildren = (node) => {
      if (!node.children) return 0
      return node.children.length + node.children.reduce((sum, child) => sum + countChildren(child), 0)
    }
  
    return countChildren(props.person)
  })
  
  const icCost = computed(() => {
    const sumIC = (node) => {
      if (!node.children || node.children.length === 0) return node.salary
      return node.children.reduce((sum, child) => sum + sumIC(child), 0)
    }
    return sumIC(props.person)
  })
  
  const managementCost = computed(() => {
    const sumManagers = (node) => {
      if (!node.children || node.children.length === 0) return 0
      const self = node.salary
      const fromChildren = node.children.reduce((sum, child) => sum + sumManagers(child), 0)
      return self + fromChildren
    }
    return sumManagers(props.person)
  })
  
  const totalCost = computed(() => {
    return icCost.value + managementCost.value
  })
  
  const ratio = computed(() => {
    const man = managementCost.value
    const ic = icCost.value
    if (man === 0) return '∞'
    return (ic / man).toFixed(2)
  })
  
  // 💡 Color logic: match department, fallback to neutral
  const cardColor = computed(() => {
    const title = props.person.position?.toLowerCase() || ''
  
    if (title.includes('engineering')) return 'bg-blue-50'
    if (title.includes('marketing')) return 'bg-pink-50'
    if (title.includes('sales')) return 'bg-yellow-50'
    if (title.includes('finance') || title.includes('accounting')) return 'bg-green-50'
    if (title.includes('hr') || title.includes('people')) return 'bg-purple-50'
    if (title.includes('product')) return 'bg-orange-50'
    if (title.includes('operations')) return 'bg-gray-100'
  
    return 'bg-white'
  })
  </script>
  
  