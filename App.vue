<template>

  <div>
    <button @click="toggleEditMode">{{ editMode ? '退出编辑模式' : '进入编辑模式' }}</button>

      <div>
          <div class="layoutJSON">
              Displayed as <code>[x, y, w, h]</code>:
              <div class="columns">
                  <div class="layoutItem" v-for="item in layout">
                      <b>{{ item.i }}</b>: [{{ item.x }}, {{ item.y }}, {{ item.w }}, {{ item.h }}]
                  </div>
              </div>
          </div>
      </div>
      <br/>
      <div @drag="drag" @dragend="dragend" class="droppable-element" draggable="true"
           unselectable="on">Droppable Element (Drag me!)</div>
      <div v-if='editMode' id="content">
          <grid-layout ref="gridlayout" :layout.sync="layout"
                       :col-num="12"
                       :row-height="30"
                       :is-draggable="true"
                       :is-resizable="true"
                       :vertical-compact="true"
                       :use-css-transforms="true"
          >
              <grid-item :key="item.i" v-for="item in layout"
                         :x="item.x"
                         :y="item.y"
                         :w="item.w"
                         :h="item.h"
                         :i="item.i"
                         ref="gridItemRefs"
              >
                  <span class="text">{{ item.i }}</span>
              </grid-item>
          </grid-layout>
      </div>
      <div v-else>
  <!-- 非编辑模式下的内容展示 -->
  <div v-for="item in layout" :key="item.i" :style="{ 
    position: 'absolute', 
    left: `${item.x * colWidth}px`, 
    top: `${item.y * rowHeight}px`,
    width: `${item.w * colWidth}px`,
    height: `${item.h * rowHeight}px`
  }">
    <span class="text">{{ item.i }}</span>
  </div>
</div>
  </div>
</template>

<script setup>
import { ref, onMounted, unref, getCurrentInstance, defineExpose } from 'vue'
let  componentInfo=null;
const ctx = getCurrentInstance().ctx
let mouseXY = { x: null, y: null }
let DragPos = { x: null, y: null, w: 1, h: 1, i: null }
let colNum = 10
const colWidth = ref(100) // 每列的宽度
const rowHeight = ref(30) // 每行的高度
const editMode = ref(true)
function toggleEditMode() {
  editMode.value = !editMode.value
}
const gridlayout = ref(null)
const gridItemRefs = ref([])
const getGridItemRef = (el, index) => {
if (el) {
  gridItemRefs.value[index] = el
}
}

const layout = ref([
{ x: 0, y: 0, w: 10, h: 1, i: '0' },
{ x: 0, y: 0, w: 10, h: 2, i: '1' },
{ x: 0, y: 0, w: 2, h: 3, i: '2' },
{ x: 0, y: 0, w: 2, h: 4, i: '3' },
{ x: 0, y: 0, w: 2, h: 5, i: '4' },
{ x: 0, y: 0, w: 2, h: 6, i: '5' },
])

onMounted(() => {
document.addEventListener(
  'dragover',
  function (e) {
    mouseXY.x = e.clientX
    mouseXY.y = e.clientY
  },
  false,
)
})

function removeItem(val) {
const layoutUnref = unref(layout)
const index = layoutUnref.map((item) => item.i).indexOf(val)
this.layout.splice(index, 1)
}

function clickGridItem(item) {
console.log(item)
}

function drag() {
const layoutUnref = unref(layout)
let parentRect = document.getElementById('content').getBoundingClientRect()
let mouseInGrid = false
if (
  mouseXY.x > parentRect.left &&
  mouseXY.x < parentRect.right &&
  mouseXY.y > parentRect.top &&
  mouseXY.y < parentRect.bottom
) {
  mouseInGrid = true
}
if (mouseInGrid === true && layoutUnref.findIndex((item) => item.i === 'drop') === -1) {
  layoutUnref.push({
    x: (layoutUnref.length * 2) % (colNum || 12),
    y: layoutUnref.length + (colNum || 12), // puts it at the bottom
    w: 1,
    h: 1,
    i: 'drop',
  })
}
let index = layoutUnref.findIndex((item) => item.i === 'drop')
if (index !== -1) {
  try {
    gridItemRefs.value[layoutUnref.length].style.display = 'none'
  } catch {}
  let el = gridItemRefs.value[index]

  if (mouseInGrid === true && el) {
    el.dragging = { top: mouseXY.y - parentRect.top, left: mouseXY.x - parentRect.left }

    let new_pos = el.calcXY(mouseXY.y - parentRect.top, mouseXY.x - parentRect.left)
    gridlayout.value.dragEvent('dragstart', 'drop', new_pos.x, new_pos.y, 1, 1)
    DragPos.i = String(index)
    DragPos.x = layoutUnref[index].x
    DragPos.y = layoutUnref[index].y
  }
  if (mouseInGrid === false) {
    gridlayout.value.dragEvent('dragend', 'drop', 1, 1, 1, 1)
    layout.value = layoutUnref.filter((obj) => obj.i !== 'drop')
  }
}
}
function dragend() {
const layoutUnref = unref(layout)
let parentRect = document.getElementById('content').getBoundingClientRect()
let mouseInGrid = false
if (
  mouseXY.x > parentRect.left &&
  mouseXY.x < parentRect.right &&
  mouseXY.y > parentRect.top &&
  mouseXY.y < parentRect.bottom
) {
  mouseInGrid = true
}
if (mouseInGrid === true) {
  gridlayout.value.dragEvent('dragend', 'drop', DragPos.x, DragPos.y, 1, 1)
  layout.value = layoutUnref.filter((obj) => obj.i !== 'drop')

  layout.value.push({
    x: DragPos.x,
    y: DragPos.y,
    w: componentInfo?.w ?? 1,
    h: componentInfo?.h ?? 1,
    i: DragPos.i,
  })
  console.log(layout.value)
  // gridlayout.value.dragEvent('dragend', DragPos.i, DragPos.x, DragPos.y, 1, 1)
  // try {
  //   gridItemRefs.value[layoutUnref.length].style.display = 'block'
  // } catch {}
}
}

defineExpose({
drag,
dragend,
})
</script>

<style scoped>

.droppable-element {
  width: 150px;
  text-align: center;
  background: #fdd;
  border: 1px solid black;
  margin: 10px 0;
  padding: 10px;
}


.vue-grid-layout {
  background: #eee;
}

.vue-grid-item:not(.vue-grid-placeholder) {
  background: #ccc;
  border: 1px solid black;
}

.vue-grid-item .resizing {
  opacity: 0.9;
}

.vue-grid-item .static {
  background: #cce;
}

.vue-grid-item .text {
  font-size: 24px;
  text-align: center;
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
  height: 100%;
  width: 100%;
}

.vue-grid-item .no-drag {
  height: 100%;
  width: 100%;
}

.vue-grid-item .minMax {
  font-size: 12px;
}

.vue-grid-item .add {
  cursor: pointer;
}

.vue-draggable-handle {
  position: absolute;
  width: 20px;
  height: 20px;
  top: 0;
  left: 0;
  background: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='10' height='10'><circle cx='5' cy='5' r='5' fill='#999999'/></svg>") no-repeat;
  background-position: bottom right;
  padding: 0 8px 8px 0;
  background-repeat: no-repeat;
  background-origin: content-box;
  box-sizing: border-box;
  cursor: pointer;
}

.layoutJSON {
  background: #ddd;
  border: 1px solid black;
  margin-top: 10px;
  padding: 10px;
}

.layoutJSON {
  background: #ddd;
  border: 1px solid black;
  margin-top: 10px;
  padding: 10px;
}

.columns {
  -moz-columns: 120px;
  -webkit-columns: 120px;
  columns: 120px;
}
</style>