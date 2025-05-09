<template>
  <div :class="['rounded-lg border p-4 shadow-md transition-all duration-200', cardColor, 'hover:shadow-lg hover:scale-[1.01]']">
    <!-- Header: Avatar, Name, Position -->
    <div class="flex justify-between items-start">
      <div class="flex gap-3 items-center">
        <div class="flex items-center justify-center w-10 h-10 rounded-full bg-gray-300 text-gray-800 font-bold dark:bg-gray-600 dark:text-white">
          {{ initials }}
        </div>
        <div>
          <h2 class="font-bold text-lg text-gray-900 dark:text-white">{{ person.name }}</h2>
          <p class="text-sm text-gray-600 italic dark:text-gray-300">{{ person.position }}</p>
        </div>
      </div>

      <!-- Collapse / Expand toggle -->
      <button
        v-if="person.children?.length"
        @click="localExpanded = !localExpanded"
        class="text-xs px-2 py-1 mt-1 bg-white text-gray-700 border rounded hover:bg-gray-100 dark:bg-gray-800 dark:text-gray-200 dark:border-gray-600"
      >
        {{ localExpanded ? 'Collapse' : 'Expand' }}
      </button>
    </div>

    <!-- Tags: Descendants, Layer, Region -->
    <div class="mt-3 flex flex-wrap gap-2 text-xs">
      <span class="bg-gray-100 dark:bg-gray-700 text-gray-800 dark:text-gray-200 px-2 py-0.5 rounded-full">
        Descendants: {{ descendantCount }}
      </span>
      <span v-if="layerTag" class="bg-blue-100 dark:bg-blue-800 text-blue-800 dark:text-blue-100 px-2 py-0.5 rounded-full">
        Layer: {{ layerTag }}
      </span>
      <span v-if="regionTag" class="bg-green-100 dark:bg-green-800 text-green-800 dark:text-green-100 px-2 py-0.5 rounded-full">
        {{ regionTag }}
      </span>
    </div>

    <!-- Cost breakdown and ratios -->
    <div class="mt-4 text-sm text-gray-800 dark:text-gray-200 space-y-1">
      <p>💰 Salary: {{ format(person.salary) }}</p>
      <p>🧑‍💼 IC Cost: {{ format(icCost) }}</p>
      <p>👨‍💼 Manager Cost: {{ format(managementCost) }}</p>
      <p>💼 Total Cost: {{ format(totalCost) }}</p>
      <p>📊 Manager:IC Ratio: {{ ratioDisplay }}</p>
      <div class="h-2 w-full bg-gray-200 rounded dark:bg-gray-700">
        <div
          class="h-2 bg-blue-500 rounded"
          :style="{ width: ratioBarWidth }"
        ></div>
      </div>
    </div>

    <!-- Recursive rendering of child nodes -->
    <div
      v-if="localExpanded && person.children?.length && depth < maxDepth"
      class="ml-6 mt-4 border-l-2 border-gray-300 pl-4 space-y-3 dark:border-gray-600"
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

// Props: person node and current tree depth
const props = defineProps({
  person: Object,
  depth: { type: Number, default: 0 }
})

// Set max visible recursion depth
const maxDepth = 3
const localExpanded = ref(true)

// Format salary numbers for readability
const format = (num) =>
  '$' + Number(num).toLocaleString(undefined, { maximumFractionDigits: 0 })

// Compute person's initials for avatar bubble
const initials = computed(() =>
  props.person.name?.split(' ').map(w => w[0]).join('').toUpperCase()
)

// Compute total descendant count (recursive)
const descendantCount = computed(() => {
  if (!props.person.children || props.person.children.length === 0) return 0
  const countChildren = (node) => {
    if (!node.children) return 0
    return node.children.length + node.children.reduce((sum, child) => sum + countChildren(child), 0)
  }
  return countChildren(props.person)
})

// IC cost: salary sum of all leaf nodes
const icCost = computed(() => {
  const sumIC = (node) => {
    if (!node.children || node.children.length === 0) return node.salary
    return node.children.reduce((sum, child) => sum + sumIC(child), 0)
  }
  return sumIC(props.person)
})

// Manager cost: salary sum of all nodes with children
const managementCost = computed(() => {
  const sumManagers = (node) => {
    if (!node.children || node.children.length === 0) return 0
    const self = node.salary
    const fromChildren = node.children.reduce((sum, child) => sum + sumManagers(child), 0)
    return self + fromChildren
  }
  return sumManagers(props.person)
})

// Total cost: sum of IC + manager cost
const totalCost = computed(() => icCost.value + managementCost.value)

// Ratio of IC to manager costs
const ratio = computed(() => {
  const man = managementCost.value
  const ic = icCost.value
  if (man === 0) return Infinity
  return ic / man
})

const ratioDisplay = computed(() =>
  ratio.value === Infinity ? '∞' : ratio.value.toFixed(2)
)

const ratioBarWidth = computed(() =>
  ratio.value === Infinity ? '100%' : Math.min(100, ratio.value * 100) + '%'
)

// Background color based on department keywords
const cardColor = computed(() => {
  const title = props.person.position?.toLowerCase() || ''
  if (title.includes('engineering')) return 'bg-blue-50 dark:bg-blue-900'
  if (title.includes('marketing')) return 'bg-pink-50 dark:bg-pink-900'
  if (title.includes('sales')) return 'bg-yellow-50 dark:bg-yellow-900'
  if (title.includes('finance') || title.includes('accounting')) return 'bg-green-50 dark:bg-green-900'
  if (title.includes('hr') || title.includes('people')) return 'bg-purple-50 dark:bg-purple-900'
  if (title.includes('product')) return 'bg-orange-50 dark:bg-orange-900'
  if (title.includes('operations')) return 'bg-gray-200 dark:bg-gray-800'
  return 'bg-white dark:bg-gray-700'
})

// Region tag extraction from location
const regionTag = computed(() => {
  const location = props.person.location || props.person["Location"]
  if (!location) return null
  const region = location.split(',').pop()?.trim()
  return region ? `Region: ${region}` : null
})

// Optional hierarchical layer classification
const layerTag = computed(() => {
  const title = props.person.position?.toLowerCase() || ''
  if (title.includes('chief')) return '1'
  if (title.includes('senior')) return '2'
  if (title.includes('intermediate')) return '3'
  if (title.includes('lead')) return '4'
  return null
})
</script>
