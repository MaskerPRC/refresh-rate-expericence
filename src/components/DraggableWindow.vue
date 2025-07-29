<template>
  <div 
    class="draggable-window"
    :style="{ 
      left: currentX + 'px', 
      top: currentY + 'px',
      width: currentWidth + 'px',
      height: currentHeight + 'px',
      transform: `translate3d(0, 0, 0)`
    }"
    ref="window"
  >
    <!-- 窗口标题栏 -->
    <div 
      class="window-header"
      @mousedown="startDrag"
      @touchstart="startDrag"
    >
      <div class="window-controls">
        <span class="control close"></span>
        <span class="control minimize"></span>
        <span class="control maximize"></span>
      </div>
      <div class="window-title">{{ title }}</div>
      <div class="window-info">
        <span class="fps-info">{{ actualFps }}fps</span>
      </div>
    </div>
    
    <!-- 窗口内容 -->
    <div class="window-content">
      <div class="content-area">
        <div class="demo-content">
                     <h3>拖动体验测试</h3>
           <p>位置: ({{ Math.round(currentX) }}, {{ Math.round(currentY) }})</p>
           <p>大小: {{ Math.round(currentWidth) }} × {{ Math.round(currentHeight) }}</p>
           <p>拖动距离: {{ Math.round(totalDistance) }}px</p>
          <div class="movement-indicator" :style="indicatorStyle"></div>
        </div>
      </div>
    </div>
    
    <!-- 窗口调整大小手柄 -->
    <div class="resize-handle" @mousedown="startResize" @touchstart="startResize"></div>
  </div>
</template>

<script>
export default {
  name: 'DraggableWindow',
  props: {
    id: {
      type: [Number, String],
      required: true
    },
    initialX: {
      type: Number,
      default: 100
    },
    initialY: {
      type: Number,
      default: 100
    },
    title: {
      type: String,
      default: '窗口'
    },
    refreshRate: {
      type: Number,
      default: 60
    },
    delay: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      isDragging: false,
      isResizing: false,
      currentX: this.initialX,
      currentY: this.initialY,
      currentWidth: 320,
      currentHeight: 240,
      startX: 0,
      startY: 0,
      startWidth: 0,
      startHeight: 0,
      offsetX: 0,
      offsetY: 0,
      actualFps: 60,
      lastFrameTime: 0,
      frameInterval: 1000 / 60,
      totalDistance: 0,
      lastPosition: { x: this.initialX, y: this.initialY },
      animationFrame: null,
      delayQueue: [], // 延迟队列（位置）
      resizeDelayQueue: [], // 延迟队列（大小）
      delayProcessor: null // 延迟处理器
    }
  },
  computed: {
    indicatorStyle() {
      const intensity = Math.min(this.totalDistance / 1000, 1)
      const hue = 120 - (intensity * 120) // 从绿色到红色
      return {
        background: `hsl(${hue}, 70%, 50%)`,
        transform: `scale(${0.5 + intensity * 0.5})`
      }
    }
  },
  watch: {
    refreshRate: {
      immediate: true,
      handler(newRate) {
        this.frameInterval = 1000 / Math.max(newRate, 1)
      }
    }
  },
  mounted() {
    this.startFpsMonitoring()
    this.startDelayProcessor()
    document.addEventListener('mousemove', this.onMouseMove)
    document.addEventListener('mouseup', this.stopActions)
    document.addEventListener('touchmove', this.onTouchMove, { passive: false })
    document.addEventListener('touchend', this.stopActions)
  },
  beforeUnmount() {
    document.removeEventListener('mousemove', this.onMouseMove)
    document.removeEventListener('mouseup', this.stopActions)
    document.removeEventListener('touchmove', this.onTouchMove)
    document.removeEventListener('touchend', this.stopActions)
    if (this.animationFrame) {
      cancelAnimationFrame(this.animationFrame)
    }
    if (this.delayProcessor) {
      cancelAnimationFrame(this.delayProcessor)
    }
  },
  methods: {
    startDrag(event) {
      event.preventDefault()
      this.isDragging = true
      
      const clientX = event.touches ? event.touches[0].clientX : event.clientX
      const clientY = event.touches ? event.touches[0].clientY : event.clientY
      
      this.startX = clientX - this.currentX
      this.startY = clientY - this.currentY
      
      this.$refs.window.style.zIndex = '9999'
    },
    
    onMouseMove(event) {
      if (this.isDragging) {
        this.handleMove(event.clientX, event.clientY)
      } else if (this.isResizing) {
        this.handleResize(event.clientX, event.clientY)
      }
    },
    
    onTouchMove(event) {
      event.preventDefault()
      const touch = event.touches[0]
      if (this.isDragging) {
        this.handleMove(touch.clientX, touch.clientY)
      } else if (this.isResizing) {
        this.handleResize(touch.clientX, touch.clientY)
      }
    },
    
    handleMove(clientX, clientY) {
      if (!this.isDragging) return
      
      const now = performance.now()
      if (now - this.lastFrameTime < this.frameInterval) {
        return // 限制刷新率
      }
      
      const newX = clientX - this.startX
      const newY = clientY - this.startY
      
      // 计算移动距离
      const distance = Math.sqrt(
        Math.pow(newX - this.lastPosition.x, 2) + 
        Math.pow(newY - this.lastPosition.y, 2)
      )
      this.totalDistance += distance
      this.lastPosition = { x: newX, y: newY }
      
      // 应用延迟 - 真正的响应延迟实现
      if (this.delay > 0) {
        // 将当前位置和时间加入延迟队列
        this.delayQueue.push({
          x: newX,
          y: newY,
          timestamp: now + this.delay
        })
      } else {
        this.updatePosition(newX, newY)
        this.lastFrameTime = now
      }
    },
    
    updatePosition(x, y) {
      // 边界检测
      const parentRect = this.$refs.window.parentElement.getBoundingClientRect()
      const windowRect = this.$refs.window.getBoundingClientRect()
      
      x = Math.max(0, Math.min(x, parentRect.width - windowRect.width))
      y = Math.max(0, Math.min(y, parentRect.height - windowRect.height))
      
      this.currentX = x
      this.currentY = y
      
      this.$emit('position-change', this.id, x, y)
    },
    
    stopActions() {
      if (this.isDragging) {
        this.isDragging = false
        this.$refs.window.style.zIndex = '1'
        // 清空位置延迟队列
        this.delayQueue = []
      }
      
      if (this.isResizing) {
        this.isResizing = false
        this.$refs.window.style.cursor = 'default'
        // 清空大小延迟队列
        this.resizeDelayQueue = []
      }
    },
    
    startResize(event) {
      event.preventDefault()
      event.stopPropagation()
      this.isResizing = true
      
      const clientX = event.touches ? event.touches[0].clientX : event.clientX
      const clientY = event.touches ? event.touches[0].clientY : event.clientY
      
      this.startX = clientX
      this.startY = clientY
      this.startWidth = this.currentWidth
      this.startHeight = this.currentHeight
      
      this.$refs.window.style.cursor = 'nw-resize'
    },
    
    handleResize(clientX, clientY) {
      if (!this.isResizing) return
      
      const now = performance.now()
      if (now - this.lastFrameTime < this.frameInterval) {
        return // 限制刷新率
      }
      
      const deltaX = clientX - this.startX
      const deltaY = clientY - this.startY
      
      const newWidth = Math.max(200, this.startWidth + deltaX) // 最小宽度200px
      const newHeight = Math.max(150, this.startHeight + deltaY) // 最小高度150px
      
      // 应用延迟 - 真正的调整大小延迟实现
      if (this.delay > 0) {
        // 将当前大小和时间加入延迟队列
        this.resizeDelayQueue.push({
          width: newWidth,
          height: newHeight,
          timestamp: now + this.delay
        })
      } else {
        this.updateSize(newWidth, newHeight)
        this.lastFrameTime = now
      }
    },
    
    updateSize(width, height) {
      // 边界检测 - 确保不超出父容器
      const parentRect = this.$refs.window.parentElement.getBoundingClientRect()
      const maxWidth = parentRect.width - this.currentX
      const maxHeight = parentRect.height - this.currentY
      
      width = Math.min(width, maxWidth - 20) // 留20px边距
      height = Math.min(height, maxHeight - 20)
      
      this.currentWidth = width
      this.currentHeight = height
    },
    
    startFpsMonitoring() {
      let frameCount = 0
      let lastTime = performance.now()
      
      const measureFps = (currentTime) => {
        frameCount++
        const deltaTime = currentTime - lastTime
        
        if (deltaTime >= 1000) {
          this.actualFps = Math.round(frameCount * 1000 / deltaTime)
          frameCount = 0
          lastTime = currentTime
        }
        
        this.animationFrame = requestAnimationFrame(measureFps)
      }
      
      this.animationFrame = requestAnimationFrame(measureFps)
    },
    
    startDelayProcessor() {
      const processDelayQueue = () => {
        const now = performance.now()
        
        // 处理延迟队列中到期的位置更新
        while (this.delayQueue.length > 0 && this.delayQueue[0].timestamp <= now) {
          const delayedUpdate = this.delayQueue.shift()
          this.updatePosition(delayedUpdate.x, delayedUpdate.y)
          this.lastFrameTime = now
        }
        
        // 处理延迟队列中到期的大小更新
        while (this.resizeDelayQueue.length > 0 && this.resizeDelayQueue[0].timestamp <= now) {
          const delayedResize = this.resizeDelayQueue.shift()
          this.updateSize(delayedResize.width, delayedResize.height)
          this.lastFrameTime = now
        }
        
        // 继续处理
        this.delayProcessor = requestAnimationFrame(processDelayQueue)
      }
      
      this.delayProcessor = requestAnimationFrame(processDelayQueue)
    }
  }
}
</script>

<style scoped>
.draggable-window {
  position: absolute;
  min-width: 200px;
  min-height: 150px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 12px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  overflow: hidden;
  transition: box-shadow 0.2s ease;
  will-change: transform, width, height;
}

.draggable-window:hover {
  box-shadow: 0 25px 50px rgba(0, 0, 0, 0.3);
}

.window-header {
  height: 40px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  align-items: center;
  padding: 0 15px;
  cursor: move;
  user-select: none;
}

.window-controls {
  display: flex;
  gap: 8px;
}

.control {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  cursor: pointer;
}

.control.close {
  background: #ff5f57;
}

.control.minimize {
  background: #ffbd2e;
}

.control.maximize {
  background: #28ca42;
}

.window-title {
  flex: 1;
  text-align: center;
  color: white;
  font-weight: 600;
  font-size: 14px;
}

.window-info {
  color: white;
  font-size: 12px;
  opacity: 0.8;
}

.window-content {
  height: calc(100% - 40px);
  padding: 20px;
  display: flex;
  flex-direction: column;
}

.content-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.demo-content h3 {
  color: #333;
  margin-bottom: 15px;
  font-size: 18px;
}

.demo-content p {
  color: #666;
  margin-bottom: 8px;
  font-size: 14px;
}

.movement-indicator {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  margin: 15px auto;
  transition: all 0.3s ease;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
}

.resize-handle {
  position: absolute;
  bottom: 0;
  right: 0;
  width: 20px;
  height: 20px;
  cursor: nw-resize;
  background: linear-gradient(-45deg, transparent 40%, #ccc 50%, transparent 60%);
  transition: all 0.2s ease;
}

.resize-handle:hover {
  background: linear-gradient(-45deg, transparent 30%, #4f46e5 50%, transparent 70%);
  transform: scale(1.2);
}

.resize-handle:active {
  background: linear-gradient(-45deg, transparent 30%, #4338ca 50%, transparent 70%);
}

/* 拖动时的样式 */
.draggable-window.dragging {
  transform: rotate(1deg);
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.4);
}
</style> 