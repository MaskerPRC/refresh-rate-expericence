<template>
  <div 
    class="draggable-window"
    :style="{ 
      left: currentX + 'px', 
      top: currentY + 'px',
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
          <p>当前位置: ({{ Math.round(currentX) }}, {{ Math.round(currentY) }})</p>
          <p>拖动距离: {{ Math.round(totalDistance) }}px</p>
          <div class="movement-indicator" :style="indicatorStyle"></div>
        </div>
      </div>
    </div>
    
    <!-- 窗口调整大小手柄 -->
    <div class="resize-handle" @mousedown="startResize"></div>
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
      startX: 0,
      startY: 0,
      offsetX: 0,
      offsetY: 0,
      actualFps: 60,
      lastFrameTime: 0,
      frameInterval: 1000 / 60,
      totalDistance: 0,
      lastPosition: { x: this.initialX, y: this.initialY },
      animationFrame: null,
      pendingUpdate: null
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
    document.addEventListener('mousemove', this.onMouseMove)
    document.addEventListener('mouseup', this.stopDrag)
    document.addEventListener('touchmove', this.onTouchMove, { passive: false })
    document.addEventListener('touchend', this.stopDrag)
  },
  beforeUnmount() {
    document.removeEventListener('mousemove', this.onMouseMove)
    document.removeEventListener('mouseup', this.stopDrag)
    document.removeEventListener('touchmove', this.onTouchMove)
    document.removeEventListener('touchend', this.stopDrag)
    if (this.animationFrame) {
      cancelAnimationFrame(this.animationFrame)
    }
    if (this.pendingUpdate) {
      clearTimeout(this.pendingUpdate)
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
      if (!this.isDragging) return
      this.handleMove(event.clientX, event.clientY)
    },
    
    onTouchMove(event) {
      if (!this.isDragging) return
      event.preventDefault()
      const touch = event.touches[0]
      this.handleMove(touch.clientX, touch.clientY)
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
      
      // 应用延迟
      if (this.delay > 0) {
        if (this.pendingUpdate) {
          clearTimeout(this.pendingUpdate)
        }
        
        this.pendingUpdate = setTimeout(() => {
          this.updatePosition(newX, newY)
          this.lastFrameTime = now
        }, this.delay)
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
    
    stopDrag() {
      if (!this.isDragging) return
      
      this.isDragging = false
      this.$refs.window.style.zIndex = '1'
      
      if (this.pendingUpdate) {
        clearTimeout(this.pendingUpdate)
        this.pendingUpdate = null
      }
    },
    
    startResize(event) {
      event.preventDefault()
      event.stopPropagation()
      this.isResizing = true
      // 这里可以添加调整大小的逻辑
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
    }
  }
}
</script>

<style scoped>
.draggable-window {
  position: absolute;
  width: 320px;
  height: 240px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 12px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  overflow: hidden;
  transition: box-shadow 0.2s ease;
  will-change: transform;
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
}

.resize-handle:hover {
  background: linear-gradient(-45deg, transparent 40%, #999 50%, transparent 60%);
}

/* 拖动时的样式 */
.draggable-window.dragging {
  transform: rotate(1deg);
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.4);
}
</style> 