<template>
  <div class="desktop" ref="desktop">
    <!-- 桌面背景网格 -->
    <div class="desktop-grid"></div>
    
    <!-- 性能信息显示 -->
    <div class="performance-info">
      <div class="info-item">
        <span class="label">当前FPS:</span>
        <span class="value">{{ currentFps }}</span>
      </div>
      <div class="info-item">
        <span class="label">目标刷新率:</span>
        <span class="value">{{ refreshRate }}Hz</span>
      </div>
      <div class="info-item">
        <span class="label">输入延迟:</span>
        <span class="value">{{ delay }}ms</span>
      </div>
    </div>
    
    <!-- 拖动的虚拟窗口 -->
    <DraggableWindow 
      v-for="window in windows"
      :key="window.id"
      :id="window.id"
      :initial-x="window.x"
      :initial-y="window.y"
      :title="window.title"
      :refresh-rate="refreshRate"
      :delay="delay"
      @position-change="updateWindowPosition"
    />
    
    <!-- 添加窗口按钮 -->
    <div class="add-window-btn" @click="addWindow">
      <span>+</span>
      <span>添加窗口</span>
    </div>
  </div>
</template>

<script>
import DraggableWindow from './DraggableWindow.vue'

export default {
  name: 'Desktop',
  components: {
    DraggableWindow
  },
  props: {
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
      windows: [
        { id: 1, x: 450, y: 120, title: '测试窗口 1' },
        { id: 2, x: 650, y: 300, title: '测试窗口 2' }
      ],
      currentFps: 60,
      frameCount: 0,
      lastTime: performance.now(),
      animationFrame: null
    }
  },
  mounted() {
    this.startFpsMonitoring()
  },
  beforeUnmount() {
    if (this.animationFrame) {
      cancelAnimationFrame(this.animationFrame)
    }
  },
  methods: {
    addWindow() {
      const desktopRect = this.$refs.desktop.getBoundingClientRect()
      const controlPanelWidth = 400 // 控制面板宽度
      const availableWidth = desktopRect.width - controlPanelWidth - 350 // 减去窗口宽度和边距
      const newWindow = {
        id: Date.now(),
        x: Math.random() * availableWidth + controlPanelWidth + 20, // 从控制面板右侧开始
        y: Math.random() * (desktopRect.height - 300) + 80, // 避开顶部
        title: `窗口 ${this.windows.length + 1}`
      }
      this.windows.push(newWindow)
    },
    updateWindowPosition(windowId, x, y) {
      const window = this.windows.find(w => w.id === windowId)
      if (window) {
        window.x = x
        window.y = y
      }
    },
    startFpsMonitoring() {
      const measureFps = (currentTime) => {
        this.frameCount++
        const deltaTime = currentTime - this.lastTime
        
        if (deltaTime >= 1000) {
          this.currentFps = Math.round(this.frameCount * 1000 / deltaTime)
          this.frameCount = 0
          this.lastTime = currentTime
        }
        
        this.animationFrame = requestAnimationFrame(measureFps)
      }
      
      this.animationFrame = requestAnimationFrame(measureFps)
    }
  }
}
</script>

<style scoped>
.desktop {
  position: relative;
  width: 100%;
  height: 100vh;
  background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
  overflow: hidden;
}

.desktop-grid {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: 
    linear-gradient(rgba(255, 255, 255, 0.05) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255, 255, 255, 0.05) 1px, transparent 1px);
  background-size: 50px 50px;
  pointer-events: none;
}

.performance-info {
  position: absolute;
  top: 20px;
  right: 20px;
  background: rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(10px);
  border-radius: 10px;
  padding: 15px;
  color: white;
  font-size: 14px;
  min-width: 200px;
  z-index: 1000;
}

.info-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
}

.info-item:last-child {
  margin-bottom: 0;
}

.label {
  color: rgba(255, 255, 255, 0.8);
}

.value {
  font-weight: bold;
  color: #4ade80;
}

.add-window-btn {
  position: absolute;
  bottom: 30px;
  right: 30px;
  background: rgba(79, 70, 229, 0.9);
  color: white;
  padding: 12px 20px;
  border-radius: 25px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 255, 255, 0.1);
}

.add-window-btn:hover {
  background: rgba(79, 70, 229, 1);
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(79, 70, 229, 0.3);
}

.add-window-btn span:first-child {
  font-size: 18px;
  font-weight: bold;
}
</style> 