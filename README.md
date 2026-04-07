# Vue Flow Demo

使用 [Vue Flow](https://vueflow.dev/) 建立的互動式流程圖展示專案。

## 功能

- 可拖曳的流程節點（開始、資料輸入、合併結果、結束等）
- **並行處理區塊**（Parallel Group）：將多個任務視覺化地分組
- 拖曳節點進出群組，自動偵測歸屬並調整位置
- 群組自動擴展以容納新拖入的子節點
- 內建背景網格與縮放控制列

## 技術棧

- Vue 3 + Composition API (`<script setup>`)
- Vue Flow (`@vue-flow/core`, `@vue-flow/background`, `@vue-flow/controls`)
- Vite

## 快速開始

```bash
npm install
npm run dev
```

開啟 http://localhost:5173 即可瀏覽。

## 建置

```bash
npm run build
npm run preview
```
