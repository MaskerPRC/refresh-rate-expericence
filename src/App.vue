<template>
  <div class="app">
    <!-- 控制面板 -->
    <ControlPanel 
      :refresh-rate="refreshRate"
      :delay="delay"
      :max-refresh-rate="maxRefreshRate"
      @update-refresh-rate="updateRefreshRate"
      @update-delay="updateDelay"
    />
    
    <!-- 模拟桌面 -->
    <Desktop 
      :refresh-rate="refreshRate"
      :delay="delay"
    />
  </div>
</template>

<script>
import Desktop from './components/Desktop.vue'
import ControlPanel from './components/ControlPanel.vue'

export default {
  name: 'App',
  components: {
    Desktop,
    ControlPanel
  },
  data() {
    return {
      refreshRate: 60, // 默认60Hz
      delay: 0, // 默认0ms延迟
      maxRefreshRate: 60 // 将在mounted中检测
    }
  },
  mounted() {
    this.detectMaxRefreshRate()
  },
  methods: {
    updateRefreshRate(rate) {
      this.refreshRate = rate
    },
    updateDelay(delay) {
      this.delay = delay
    },
    detectMaxRefreshRate() {
      // 检测屏幕最大刷新率
      let lastTime = performance.now()
      let frameCount = 0
      const samples = []
      
      const measure = (currentTime) => {
        const deltaTime = currentTime - lastTime
        if (deltaTime > 0) {
          samples.push(1000 / deltaTime)
        }
        lastTime = currentTime
        frameCount++
        
        if (frameCount < 60) {
          requestAnimationFrame(measure)
        } else {
          // 计算平均刷新率
          const avgRefreshRate = samples.reduce((a, b) => a + b, 0) / samples.length
          this.maxRefreshRate = Math.min(Math.round(avgRefreshRate), 240) // 最高240Hz
        }
      }
      
      requestAnimationFrame(measure)
    }
  }
}
</script>

<style>
.app {
  height: 100vh;
  width: 100vw;
  position: relative;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}
</style> 