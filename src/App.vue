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
      maxRefreshRate: 240 // 固定最大240Hz
    }
  },
  mounted() {
    // 可选：尝试获取屏幕真实刷新率
    this.tryGetScreenRefreshRate()
  },
  methods: {
    updateRefreshRate(rate) {
      this.refreshRate = rate
    },
    updateDelay(delay) {
      this.delay = delay
    },
    tryGetScreenRefreshRate() {
      // 尝试使用Screen API获取屏幕信息（实验性API）
      if ('screen' in window && 'orientation' in window.screen) {
        try {
          // 某些浏览器支持获取屏幕信息
          console.log('屏幕信息:', {
            width: screen.width,
            height: screen.height,
            colorDepth: screen.colorDepth,
            pixelDepth: screen.pixelDepth
          })
          
          // 检查是否支持Display API
          if ('getDisplayMedia' in navigator.mediaDevices) {
            console.log('浏览器支持Display API')
          }
        } catch (error) {
          console.log('无法获取详细屏幕信息:', error)
        }
      }
      
      // 使用requestAnimationFrame简单估算（但固定上限为240）
      let frameCount = 0
      let lastTime = performance.now()
      
      const quickCheck = (currentTime) => {
        const deltaTime = currentTime - lastTime
        if (deltaTime > 0 && frameCount < 10) {
          const currentFps = Math.round(1000 / deltaTime)
          if (currentFps > 200) {
            console.log(`检测到高刷新率: ${currentFps}Hz`)
          }
        }
        lastTime = currentTime
        frameCount++
        
        if (frameCount < 10) {
          requestAnimationFrame(quickCheck)
        }
      }
      
      requestAnimationFrame(quickCheck)
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