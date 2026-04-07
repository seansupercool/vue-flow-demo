<script setup>
import { ref } from 'vue'
import { VueFlow, useVueFlow } from '@vue-flow/core'
import { Background } from '@vue-flow/background'
import { Controls } from '@vue-flow/controls'
import ParallelGroupNode from './ParallelGroupNode.vue'
import ProcessNode from './ProcessNode.vue'

const nodeTypes = {
  parallelGroup: ParallelGroupNode,
  process: ProcessNode,
}

const { findNode, updateNode, getIntersectingNodes, onNodeDragStop } = useVueFlow()

// 流程節點定義
const nodes = ref([
  {
    id: 'start',
    type: 'process',
    position: { x: 300, y: 0 },
    data: { label: '開始' },
  },
  {
    id: 'step1',
    type: 'process',
    position: { x: 300, y: 100 },
    data: { label: '資料輸入' },
  },

  // === 多序區塊（並行處理）===
  {
    id: 'parallel-group-1',
    type: 'parallelGroup',
    position: { x: 120, y: 210 },
    data: { label: '並行處理區' },
    style: { width: '480px', height: '160px' },
  },
  // 任務 A 在群組內 → border 紅色
  {
    id: 'parallel-a',
    type: 'process',
    position: { x: 30, y: 50 },
    data: { label: '任務 A：驗證' },
    parentNode: 'parallel-group-1',
  },
  // 任務 B 在群組外 → border 預設紫色，拖進群組後變紅色
  {
    id: 'parallel-b',
    type: 'process',
    position: { x: 30, y: 420 },
    data: { label: '任務 B：轉換' },
  },
  // 任務 C 在群組外
  {
    id: 'parallel-c',
    type: 'process',
    position: { x: 500, y: 420 },
    data: { label: '任務 C：通知' },
  },

  // 合併後的步驟
  {
    id: 'merge',
    type: 'process',
    position: { x: 300, y: 420 },
    data: { label: '合併結果' },
  },

  // === 第二個多序區塊 ===
  {
    id: 'parallel-group-2',
    type: 'parallelGroup',
    position: { x: 180, y: 530 },
    data: { label: '並行審核區' },
    style: { width: '360px', height: '160px' },
  },
  {
    id: 'review-a',
    type: 'process',
    position: { x: 20, y: 40 },
    data: { label: '主管審核' },
    parentNode: 'parallel-group-2',
  },
  {
    id: 'review-b',
    type: 'process',
    position: { x: 195, y: 40 },
    data: { label: '風控審核' },
    parentNode: 'parallel-group-2',
  },

  // 結束
  {
    id: 'end',
    type: 'process',
    position: { x: 300, y: 740 },
    data: { label: '結束' },
  },
])

const edges = ref([
  { id: 'e-start-step1', source: 'start', target: 'step1', animated: true },
  { id: 'e-step1-group1', source: 'step1', target: 'parallel-group-1' },
  { id: 'e-group1-merge', source: 'parallel-group-1', target: 'merge' },
  { id: 'e-merge-group2', source: 'merge', target: 'parallel-group-2' },
  { id: 'e-group2-end', source: 'parallel-group-2', target: 'end' },
])

// 拖曳結束時，檢查是否放進/拖出群組
onNodeDragStop(({ node }) => {
  if (node.type === 'parallelGroup') return

  const graphNode = findNode(node.id)
  if (!graphNode) return

  const currentParent = graphNode.parentNode
  const parentGroup = currentParent ? findNode(currentParent) : null

  // 計算節點的絕對位置
  const absolutePos = parentGroup
    ? { x: parentGroup.position.x + graphNode.position.x, y: parentGroup.position.y + graphNode.position.y }
    : { ...graphNode.position }

  const nodeW = graphNode.dimensions?.width || 150
  const nodeH = graphNode.dimensions?.height || 50

  // 用節點中心點判斷是否在群組內（避免只碰到邊就觸發）
  const centerX = absolutePos.x + nodeW / 2
  const centerY = absolutePos.y + nodeH / 2

  const allGroups = nodes.value.filter((n) => n.type === 'parallelGroup')
  const targetGroup = allGroups.find((group) => {
    const gNode = findNode(group.id)
    if (!gNode) return false
    const gx = gNode.position.x
    const gy = gNode.position.y
    const gw = gNode.dimensions?.width || parseInt(gNode.style?.width) || 300
    const gh = gNode.dimensions?.height || parseInt(gNode.style?.height) || 160
    return centerX > gx && centerX < gx + gw && centerY > gy && centerY < gy + gh
  })

  const targetGroupNode = targetGroup ? findNode(targetGroup.id) : null

  if (targetGroupNode && currentParent !== targetGroupNode.id) {
    // 放進群組
    const relX = absolutePos.x - targetGroupNode.position.x
    const relY = absolutePos.y - targetGroupNode.position.y
    updateNode(graphNode.id, {
      parentNode: targetGroupNode.id,
      position: { x: Math.max(10, relX), y: Math.max(30, relY) },
    })
    expandGroupToFit(targetGroupNode, Math.max(10, relX), Math.max(30, relY), nodeW, nodeH)
  } else if (!targetGroupNode && currentParent) {
    // 拖出群組
    updateNode(graphNode.id, {
      parentNode: '',
      position: absolutePos,
    })
  }
})

// 自動撐大群組框，確保完整包住子節點 + padding
function expandGroupToFit(group, childX, childY, childW, childH) {
  const padding = 20
  const currentWidth = parseInt(group.style?.width) || 300
  const currentHeight = parseInt(group.style?.height) || 160

  const neededWidth = childX + childW + padding
  const neededHeight = childY + childH + padding

  const newWidth = Math.max(currentWidth, neededWidth)
  const newHeight = Math.max(currentHeight, neededHeight)

  if (newWidth > currentWidth || newHeight > currentHeight) {
    updateNode(group.id, {
      style: {
        ...group.style,
        width: `${newWidth}px`,
        height: `${newHeight}px`,
      },
    })
  }
}
</script>

<template>
  <div class="flow-container">
    <VueFlow
      :nodes="nodes"
      :edges="edges"
      :node-types="nodeTypes"
      :default-viewport="{ zoom: 1, x: 50, y: 20 }"
      fit-view-on-init
    >
      <Background />
      <Controls />
    </VueFlow>
  </div>
</template>

<style scoped>
.flow-container {
  width: 100%;
  height: 100vh;
}
</style>
