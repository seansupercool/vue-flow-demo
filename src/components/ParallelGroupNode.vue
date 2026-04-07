<script setup>
import { computed } from 'vue'
import { Handle, Position, useVueFlow } from '@vue-flow/core'

const props = defineProps({
  id: { type: String, required: true },
  data: { type: Object, required: true },
})

const { getIntersectingNodes, getNodes } = useVueFlow()

// 檢查是否有節點正在被拖曳且靠近此群組
const isDropTarget = computed(() => {
  const draggingNodes = getNodes.value.filter((n) => n.dragging && n.type !== 'parallelGroup')
  if (draggingNodes.length === 0) return false

  return draggingNodes.some((dragNode) => {
    const intersections = getIntersectingNodes(dragNode)
    return intersections.some((n) => n.id === props.id)
  })
})
</script>

<template>
  <div class="parallel-group-node" :class="{ 'drop-target': isDropTarget }">
    <Handle type="target" :position="Position.Top" />
    <div class="group-label">{{ data.label }}</div>
    <Handle type="source" :position="Position.Bottom" />
  </div>
</template>

<style scoped>
.parallel-group-node {
  width: 100%;
  height: 100%;
  background: rgba(59, 130, 246, 0.05);
  border: 2px dashed #3b82f6;
  border-radius: 12px;
  padding: 10px;
  box-sizing: border-box;
  pointer-events: all;
  transition: background 0.2s, border-color 0.2s, box-shadow 0.2s;
}

.parallel-group-node.drop-target {
  background: rgba(59, 130, 246, 0.15);
  border-color: #2563eb;
  box-shadow: 0 0 16px rgba(59, 130, 246, 0.3);
}

.group-label {
  font-size: 12px;
  font-weight: 600;
  color: #3b82f6;
  text-align: center;
  padding: 2px 8px;
}
</style>
