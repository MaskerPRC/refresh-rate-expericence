<template>
  <div class="control-panel" :class="{ 'collapsed': isCollapsed }">
    <!-- 折叠/展开按钮 -->
    <div class="toggle-btn" @click="togglePanel">
      <span class="icon">{{ isCollapsed ? '⚙️' : '✕' }}</span>
    </div>
    
    <!-- 面板内容 -->
    <div class="panel-content" v-show="!isCollapsed">
      <div class="panel-header">
        <h2 class="panel-title">🎮 刷新率体验控制台</h2>
        <p class="panel-subtitle">调整参数体验不同的性能表现</p>
      </div>
      
      <!-- 刷新率控制 -->
      <div class="control-group">
        <div class="control-header">
          <label class="control-label">
            <span class="icon">📺</span>
            刷新率控制
          </label>
          <div class="control-value">
            {{ refreshRate }}Hz
          </div>
        </div>
        
        <div class="slider-container">
          <input 
            type="range" 
            class="slider refresh-rate-slider"
            :min="5" 
            :max="maxRefreshRate"
            :value="refreshRate"
            @input="updateRefreshRate"
            @change="updateRefreshRate"
          >
          <div class="slider-marks">
            <span class="mark" style="left: 0%">5Hz</span>
            <span class="mark" style="left: 25%">30Hz</span>
            <span class="mark" style="left: 50%">60Hz</span>
            <span class="mark" style="left: 75%">120Hz</span>
            <span class="mark" style="left: 100%">{{ maxRefreshRate }}Hz</span>
          </div>
        </div>
        
        <div class="preset-buttons">
          <button 
            v-for="preset in refreshRatePresets"
            :key="preset"
            class="preset-btn"
            :class="{ active: refreshRate === preset }"
            @click="setRefreshRate(preset)"
          >
            {{ preset }}Hz
          </button>
        </div>
      </div>
      
      <!-- 延迟控制 -->
      <div class="control-group">
        <div class="control-header">
          <label class="control-label">
            <span class="icon">⏱️</span>
            输入延迟模拟
          </label>
          <div class="control-value">
            {{ delay }}ms
          </div>
        </div>
        
        <div class="slider-container">
          <input 
            type="range" 
            class="slider delay-slider"
            :min="0" 
            :max="1000"
            :step="10"
            :value="delay"
            @input="updateDelay"
            @change="updateDelay"
          >
          <div class="slider-marks">
            <span class="mark" style="left: 0%">0ms</span>
            <span class="mark" style="left: 25%">250ms</span>
            <span class="mark" style="left: 50%">500ms</span>
            <span class="mark" style="left: 75%">750ms</span>
            <span class="mark" style="left: 100%">1s</span>
          </div>
        </div>
        
        <div class="preset-buttons">
          <button 
            v-for="preset in delayPresets"
            :key="preset"
            class="preset-btn"
            :class="{ active: delay === preset }"
            @click="setDelay(preset)"
          >
            {{ preset }}ms
          </button>
        </div>
      </div>
      
      <!-- 信息提示 -->
      <div class="info-section">
        <div class="info-card">
          <h4>💡 使用提示</h4>
          <ul>
            <li>拖动桌面上的窗口感受不同刷新率的差异</li>
            <li>高刷新率 (120Hz+) 提供更流畅的拖动体验</li>
            <li>延迟模拟可以体验不同网络条件下的响应</li>
            <li>观察窗口内的运动指示器颜色变化</li>
          </ul>
        </div>
        
        <div class="stats-card">
          <h4>📊 当前状态</h4>
          <div class="stat-item">
            <span class="stat-label">屏幕最大刷新率:</span>
            <span class="stat-value">{{ maxRefreshRate }}Hz</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">性能等级:</span>
            <span class="stat-value" :class="performanceClass">{{ performanceLevel }}</span>
          </div>
        </div>
      </div>
      
      <!-- 重置按钮 -->
      <div class="action-buttons">
        <button class="btn btn-secondary" @click="resetToDefaults">
          🔄 重置默认
        </button>
        <button class="btn btn-primary" @click="optimizeSettings">
          ⚡ 自动优化
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ControlPanel',
  props: {
    refreshRate: {
      type: Number,
      default: 60
    },
    delay: {
      type: Number,
      default: 0
    },
    maxRefreshRate: {
      type: Number,
      default: 60
    }
  },
  data() {
    return {
      isCollapsed: false,
      refreshRatePresets: [30, 60, 90, 120, 144],
      delayPresets: [0, 50, 100, 200, 500]
    }
  },
  computed: {
    performanceLevel() {
      if (this.refreshRate >= 120 && this.delay <= 50) {
        return '高性能'
      } else if (this.refreshRate >= 60 && this.delay <= 100) {
        return '平衡'
      } else {
        return '节能模式'
      }
    },
    performanceClass() {
      const level = this.performanceLevel
      if (level === '高性能') return 'performance-high'
      if (level === '平衡') return 'performance-medium'
      return 'performance-low'
    }
  },
  methods: {
    togglePanel() {
      this.isCollapsed = !this.isCollapsed
    },
    updateRefreshRate(event) {
      const value = parseInt(event.target.value)
      this.$emit('update-refresh-rate', value)
    },
    updateDelay(event) {
      const value = parseInt(event.target.value)
      this.$emit('update-delay', value)
    },
    setRefreshRate(rate) {
      this.$emit('update-refresh-rate', rate)
    },
    setDelay(delay) {
      this.$emit('update-delay', delay)
    },
    resetToDefaults() {
      this.$emit('update-refresh-rate', 60)
      this.$emit('update-delay', 0)
    },
    optimizeSettings() {
      // 根据设备性能自动优化设置
      const optimalRate = Math.min(this.maxRefreshRate, 120)
      this.$emit('update-refresh-rate', optimalRate)
      this.$emit('update-delay', 0)
    }
  }
}
</script>

<style scoped>
.control-panel {
  position: fixed;
  top: 20px;
  left: 20px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(20px);
  border-radius: 16px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  z-index: 1000;
  transition: all 0.3s ease;
  max-width: 380px;
  min-width: 320px;
}

.control-panel.collapsed {
  width: 60px;
  height: 60px;
  border-radius: 50%;
}

.toggle-btn {
  position: absolute;
  top: 15px;
  right: 15px;
  width: 30px;
  height: 30px;
  background: rgba(79, 70, 229, 0.1);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s ease;
  z-index: 1001;
}

.control-panel.collapsed .toggle-btn {
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.toggle-btn:hover {
  background: rgba(79, 70, 229, 0.2);
  transform: scale(1.1);
}

.toggle-btn .icon {
  font-size: 16px;
}

.panel-content {
  padding: 25px;
  padding-top: 50px;
}

.panel-header {
  margin-bottom: 25px;
  text-align: center;
}

.panel-title {
  color: #333;
  font-size: 20px;
  font-weight: 700;
  margin-bottom: 5px;
}

.panel-subtitle {
  color: #666;
  font-size: 14px;
  margin: 0;
}

.control-group {
  margin-bottom: 30px;
  padding: 20px;
  background: rgba(79, 70, 229, 0.05);
  border-radius: 12px;
  border: 1px solid rgba(79, 70, 229, 0.1);
}

.control-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.control-label {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 600;
  color: #333;
  font-size: 16px;
}

.control-value {
  background: #4f46e5;
  color: white;
  padding: 4px 12px;
  border-radius: 20px;
  font-weight: 600;
  font-size: 14px;
  min-width: 60px;
  text-align: center;
}

.slider-container {
  position: relative;
  margin-bottom: 15px;
}

.slider {
  width: 100%;
  height: 8px;
  border-radius: 4px;
  background: rgba(79, 70, 229, 0.1);
  outline: none;
  -webkit-appearance: none;
  appearance: none;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #4f46e5;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 2px 8px rgba(79, 70, 229, 0.3);
}

.slider::-webkit-slider-thumb:hover {
  background: #4338ca;
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(79, 70, 229, 0.4);
}

.refresh-rate-slider::-webkit-slider-thumb {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.delay-slider::-webkit-slider-thumb {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
}

.slider-marks {
  display: flex;
  justify-content: space-between;
  margin-top: 8px;
  position: relative;
}

.mark {
  font-size: 11px;
  color: #666;
  position: absolute;
  transform: translateX(-50%);
}

.preset-buttons {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.preset-btn {
  padding: 6px 12px;
  border: 2px solid rgba(79, 70, 229, 0.2);
  background: rgba(79, 70, 229, 0.05);
  color: #4f46e5;
  border-radius: 20px;
  cursor: pointer;
  font-size: 12px;
  font-weight: 600;
  transition: all 0.2s ease;
  min-width: 50px;
}

.preset-btn:hover {
  background: rgba(79, 70, 229, 0.1);
  transform: translateY(-1px);
}

.preset-btn.active {
  background: #4f46e5;
  color: white;
  border-color: #4f46e5;
}

.info-section {
  display: grid;
  grid-template-columns: 1fr;
  gap: 15px;
  margin-bottom: 25px;
}

.info-card, .stats-card {
  background: rgba(0, 0, 0, 0.03);
  padding: 15px;
  border-radius: 10px;
  border: 1px solid rgba(0, 0, 0, 0.1);
}

.info-card h4, .stats-card h4 {
  margin: 0 0 10px 0;
  color: #333;
  font-size: 14px;
  font-weight: 600;
}

.info-card ul {
  margin: 0;
  padding-left: 16px;
  color: #666;
  font-size: 12px;
  line-height: 1.5;
}

.info-card li {
  margin-bottom: 4px;
}

.stat-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
  font-size: 13px;
}

.stat-label {
  color: #666;
}

.stat-value {
  font-weight: 600;
}

.stat-value.performance-high {
  color: #059669;
}

.stat-value.performance-medium {
  color: #d97706;
}

.stat-value.performance-low {
  color: #dc2626;
}

.action-buttons {
  display: flex;
  gap: 10px;
}

.btn {
  flex: 1;
  padding: 12px 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  font-size: 14px;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.btn-secondary {
  background: rgba(107, 114, 128, 0.1);
  color: #6b7280;
  border: 2px solid rgba(107, 114, 128, 0.2);
}

.btn-secondary:hover {
  background: rgba(107, 114, 128, 0.15);
  transform: translateY(-1px);
}

.btn-primary {
  background: #4f46e5;
  color: white;
}

.btn-primary:hover {
  background: #4338ca;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(79, 70, 229, 0.3);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .control-panel {
    left: 10px;
    right: 10px;
    max-width: none;
    min-width: auto;
  }
  
  .panel-content {
    padding: 20px;
    padding-top: 45px;
  }
  
  .info-section {
    grid-template-columns: 1fr;
  }
}
</style> 