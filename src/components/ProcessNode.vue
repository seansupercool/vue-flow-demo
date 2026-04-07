<script setup>
import { Handle, Position } from '@vue-flow/core'

const props = defineProps({
  id: { type: String, required: true },
  data: { type: Object, required: true },
  parentNodeId: { type: String, default: '' },
})
</script>

<template>
  <div class="process-node" :class="{ 'in-group': !!parentNodeId }">
    <Handle type="target" :position="Position.Top" />
    <div class="node-label">{{ data.label }}</div>
    <Handle type="source" :position="Position.Bottom" />
  </div>
</template>

<style scoped>
.process-node {
  background: #ffffff;
  border: 2px solid #6366f1;
  border-radius: 8px;
  padding: 12px 20px;
  min-width: 120px;
  text-align: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: border-color 0.2s;
}

.process-node.in-group {
  border-color: #ef4444;
}

.node-label {
  font-size: 14px;
  font-weight: 500;
  color: #1e293b;
}
</style>
