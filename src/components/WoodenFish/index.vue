<script setup>
import { ref, onMounted } from 'vue';

const props = defineProps({
  bottom: {
    type: [String, Number],
    default: '100px',
  },
  right: {
    type: [String, Number],
    default: '20px',
  },
});

// 木鱼状态
const woodenFishProps = ref({
  bottom: '',
  right: '',
  isClicking: false
});

// 处理props变化
const updatePosition = () => {
  woodenFishProps.value.bottom = /^[\d|.]*$/g.test(props.bottom) ? props.bottom + 'px' : props.bottom;
  woodenFishProps.value.right = /^[\d|.]*$/g.test(props.right) ? props.right + 'px' : props.right;
};

updatePosition();

// 最简单的声音实现
const playSound = () => {
  try {
    // 使用Web Audio API直接生成声音
    const audioContext = new (window.AudioContext || window.webkitAudioContext)();
    const oscillator = audioContext.createOscillator();
    const gainNode = audioContext.createGain();

    oscillator.connect(gainNode);
    gainNode.connect(audioContext.destination);

    oscillator.type = 'sine';
    oscillator.frequency.value = 800; // 设置频率
    gainNode.gain.value = 0.1; // 设置音量

    // 创建音量淡入淡出效果
    gainNode.gain.setValueAtTime(0, audioContext.currentTime);
    gainNode.gain.linearRampToValueAtTime(0.1, audioContext.currentTime + 0.05);
    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.3);

    oscillator.start(audioContext.currentTime);
    oscillator.stop(audioContext.currentTime + 0.3); // 0.3秒后停止
  } catch (error) {
    console.log('播放声音失败:', error);
  }
};

// 点击木鱼 - 简化到极致的实现
const clickWoodenFish = (event) => {
  // 避免快速连续点击
  if (woodenFishProps.value.isClicking) return;
  woodenFishProps.value.isClicking = true;

  // 播放声音
  playSound();

  // 创建道德+1元素并添加到body
  showMoralPopup(event);

  // 重置点击状态
  setTimeout(() => {
    woodenFishProps.value.isClicking = false;
  }, 300);
};

// 显示道德+1弹出效果 - 直接操作DOM确保显示
const showMoralPopup = (event) => {
  // 创建一个新的div元素
  const popup = document.createElement('div');

  // 设置元素内容
  popup.textContent = '道德+1';

  // 获取点击位置
  const rect = event.currentTarget.getBoundingClientRect();

  // 设置内联样式，确保元素显示在最上层
  popup.style.cssText = `
    position: fixed;
    left: ${rect.left + rect.width / 2 - 25}px;
    top: ${rect.top - 50}px;
    color: #3c763d;
    font-size: 24px;
    font-weight: bold;
    text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
    z-index: 999999;
    pointer-events: none;
    animation: moralFloat 1.5s ease-out forwards;
  `;

  // 添加到body
  document.body.appendChild(popup);

  // 动画结束后移除元素
  setTimeout(() => {
    if (popup.parentNode) {
      popup.parentNode.removeChild(popup);
    }
  }, 1500);
};

// 添加动画样式到页面
const addAnimationStyle = () => {
  // 检查是否已经添加过样式
  if (!document.getElementById('wooden-fish-animation-style')) {
    const style = document.createElement('style');
    style.id = 'wooden-fish-animation-style';
    style.textContent = `
      @keyframes moralFloat {
        0% {
          opacity: 1;
          transform: translateY(0) scale(1);
        }
        50% {
          opacity: 0.9;
          transform: translateY(-30px) scale(1.3);
        }
        100% {
          opacity: 0;
          transform: translateY(-80px) scale(0.8);
        }
      }
    `;
    document.head.appendChild(style);
  }
};

onMounted(() => {
  // 添加动画样式
  addAnimationStyle();

  // 解锁音频上下文
  const unlockAudioContext = () => {
    try {
      const audioContext = new (window.AudioContext || window.webkitAudioContext)();
      audioContext.resume();
      document.removeEventListener('click', unlockAudioContext);
    } catch (e) {
      console.log('音频上下文解锁失败:', e);
    }
  };
  document.addEventListener('click', unlockAudioContext);
});
</script>

<template>
  <div class="wooden-fish-container" :style="`bottom: ${woodenFishProps.bottom}; right: ${woodenFishProps.right};`"
    @click="clickWoodenFish">
    <!-- 简单的木鱼图标 -->
    <!-- 木鱼logo占位 -->
    <div class="wooden-fish-icon">
      道德
    </div>
  </div>
</template>

<style lang="scss" scoped>
.wooden-fish-container {
  position: fixed;
  z-index: 9999;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 15px;
  background-color: rgba(222, 184, 135, 0.95);
  border-radius: 50%;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);

  &:hover {
    transform: scale(1.1);
    box-shadow: 0 6px 25px rgba(0, 0, 0, 0.4);
  }

  &:active {
    transform: scale(0.9);
  }
}

.wooden-fish-icon {
  width: 60px;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: bold;
  color: #8B4513;
  text-shadow: 1px 1px 1px rgba(255, 255, 255, 0.5);
}
</style>