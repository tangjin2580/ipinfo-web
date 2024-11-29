<template>
  <div class="history-box">
    <h2 @click="toggleVisibility" style="cursor: pointer;">查询历史记录 <span>{{ isVisible ? '▲' : '▼' }}</span></h2>
    <transition name="fade">
      <div v-if="isVisible" class="results-grid">
        <div v-for="(record, index) in history" :key="index" class="ip-info-box">
          <p>查询域名: <span>{{ record.domain }}</span></p>
          <p>查询时间: <span>{{ record.time }}</span></p>
          <p>查询信息: <strong>{{ record.ip }}</strong></p>
          <p>国家: <span>{{ record.country || '未知' }}</span></p>
          <p>城市: <span>{{ record.city || '没有记录' }}</span></p>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  props: {
    history: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      isVisible: true // 默认显示历史记录
    };
  },
  methods: {
    toggleVisibility() {
      this.isVisible = !this.isVisible; // 切换历史记录显示状态
    }
  }
};
</script>

<style scoped>
.history-box {
  position: fixed;
  right: 20px; /* 距离右侧20px */
  top: 50px; /* 距离顶部50px */
  width: 10%; /* 宽度为90%，以适应不同屏幕 */
  max-width: 175px; /* 最大宽度设置 */
  background-color: rgba(255, 255, 255, 0.9);
  border: 1px solid #ccc;
  border-radius: 10px;
  padding: 15px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  overflow-y: auto; /* 允许滚动 */
  max-height: 600px; /* 限制最大高度 */
  transition: max-height 0.5s ease-in-out; /* 动画效果 */
}

.results-grid {
  display: flex;
  flex-direction: column; /* 垂直排列 */
  gap: 10px; /* 盒子之间的间隔 */
}

.ip-info-box {
  padding: 10px;
  border: 1px solid #007BFF;
  border-radius: 5px;
  background-color: rgba(0, 123, 255, 0.1);
}

/* 渐变动画 */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active 在进入之前的状态 */ {
  opacity: 0;
}
</style>
