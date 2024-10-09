<template>
  <div class="container">
    <span class="counter">{{ counter }}</span>

    <!-- 木鱼图片和功德文字 -->
    <div class="muyu-container">
      <span class="description" style="margin-bottom: 20px;">功德</span>

      <!-- 添加棒子和木鱼 -->
      <div class="muyu-group">
        <img
            src="../assets/muyu-stick.png"
            alt="棒子"
            class="stick-icon"
            :class="{ 'stick-hit-animation': isHitting }"
        />
        <img
            src="../assets/muyu.png"
            alt="木鱼"
            class="muyu-icon"
            @click="playMuyu"
            :class="{ 'hit-animation': isHitting }"
        />
        <!-- 功德 +1 动画 -->
        <span v-show="showBonus" class="bonus-text">{{ wishText }}</span>
      </div>
    </div>

    <span class="click-text">点击木鱼或按空格增加功德</span>

    <!-- Element Plus 开关及选项 -->
    <div class="switch-options-container">
      <el-switch v-model="immersiveMode" active-text="沉浸模式"/>
      <span class="option underline" @click="openWishDIY">心愿DIY</span>
      <span class="option underline" @click="openAutoKnockDialog">自动敲</span>
    </div>

    <el-dialog width="300px" v-model="wishDialogVisible" title="心愿DIY">
      <el-radio-group v-model="selectedWish" >
        <el-radio label="功德 + 1, 诸神保佑">功德 + 1, 诸神保佑</el-radio>
        <el-radio label="平安喜乐, 功德 + 1">平安喜乐, 功德 + 1</el-radio>
        <el-radio label="烦恼 - 1, 功德 + 1">烦恼 - 1, 功德 + 1</el-radio>
        <el-radio label="custom">自定义</el-radio>
      </el-radio-group>

      <el-input style="margin-bottom: 10px" v-if="selectedWish === 'custom'" v-model="customWish" maxlength="15"
                placeholder="请输入不超过15字节的内容"/>

      <span slot="footer" >
        <el-button @click="wishDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="setWishText">确定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="自动敲" width="200px" v-model="isAutoKnockDialogVisible ">
      <el-radio-group style="margin-bottom: 10px" v-model="autoKnockOption">
        <el-radio :label="true">开启自动敲</el-radio>
        <el-radio :label="false">关闭自动敲</el-radio>
      </el-radio-group >

      <!-- 底部按钮 -->
      <span slot="footer">
        <el-button @click="cancelAutoKnock">取消</el-button>
        <el-button type="primary" @click="applyAutoKnock">应用</el-button>
      </span>
    </el-dialog>

    <audio ref="bgMusic" loop>
      <source src="../assets/background-music.mp3" type="audio/mpeg">
      Your browser does not support the audio element.
    </audio>

    <audio ref="muyuSound">
      <source src="../assets/muyu-sound.mp3" type="audio/mpeg">
      Your browser does not support the audio element.
    </audio>
  </div>
</template>

<script setup>
import {onBeforeUnmount, onMounted, ref, watch} from 'vue';

const counter = ref(0);
const immersiveMode = ref(false);
const isHitting = ref(false);
const muyuSound = ref(null);
const bgMusic = ref(null);
const isSpacePressed = ref(false);
const showBonus = ref(false);

// 心愿相关
const wishText = ref('功德 +1,漫天神佛保佑你'); // 初始显示的心愿内容
const selectedWish = ref('功德 + 1, 诸神保佑'); // 默认选择的心愿
const customWish = ref(''); // 自定义心愿
const wishDialogVisible = ref(false); // 控制弹窗的可见性

//自动敲
const isAutoKnockDialogVisible  = ref(false); // 控制自动敲弹窗的可见性
const autoKnockOption = ref(false); // 记录是否开启自动敲
const isAutoKnocking = ref(false); // 控制是否正在自动敲击
let autoKnockTimer = null; // 自动敲的定时器

// 打开自动敲弹窗
const openAutoKnockDialog  = () => {
  isAutoKnockDialogVisible.value = true;
};

// 取消按钮逻辑
const cancelAutoKnock = () => {
  isAutoKnockDialogVisible.value = false;
};

// 开始自动敲击逻辑
const startAutoKnock = () => {
  if (isAutoKnocking.value) return; // 防止重复启动
  isAutoKnocking.value = true;
  autoKnockTimer = setInterval(() => {
    playMuyu(); // 每隔1秒敲击一次木鱼
  }, 1200); // 设置敲击频率为1秒一次
};

// 停止自动敲击逻辑
const stopAutoKnock = () => {
  if (!isAutoKnocking.value) return;
  isAutoKnocking.value = false;
  clearInterval(autoKnockTimer); // 停止自动敲的定时器
};

// 应用自动敲设置
const applyAutoKnock = () => {
  if (autoKnockOption.value) {
    startAutoKnock(); // 开启自动敲
    console.log("自动敲已开启");
  } else {
    stopAutoKnock(); // 关闭自动敲
    console.log("自动敲已关闭");
  }
  isAutoKnockDialogVisible.value = false; // 关闭弹窗
};

// 清理定时器资源，防止内存泄漏
onBeforeUnmount(() => {
  stopAutoKnock(); // 组件销毁时停止自动敲
});

// 打开心愿DIY弹窗
const openWishDIY = () => {
  wishDialogVisible.value = true;
};

// 确定心愿内容
const setWishText = () => {
  wishText.value = selectedWish.value === 'custom' ? customWish.value : selectedWish.value;
  wishDialogVisible.value = false; // 关闭弹窗
};

const increment = () => {
  counter.value++;
  showBonus.value = true;
  setTimeout(() => {
    showBonus.value = false;
  }, 1000);
};

const playMuyu = () => {
  if (isHitting.value) return;
  increment();

  if (muyuSound.value) {
    muyuSound.value.currentTime = 0;
    muyuSound.value.play().catch((error) => {
      console.error('音频播放失败: ', error);
    });
  }

  isHitting.value = true;

  setTimeout(() => {
    isHitting.value = false;
  }, 100);
};

// 监听空格键事件
const handleKeyPress = (event) => {
  if (event.code === 'Space' && !isSpacePressed.value) {
    isSpacePressed.value = true;
    playMuyu();
  }
};

const handleKeyRelease = (event) => {
  if (event.code === 'Space') {
    isSpacePressed.value = false;
  }
};

watch(immersiveMode, (newValue) => {
  if (newValue) {
    bgMusic.value.play().catch((error) => {
      console.error('背景音乐播放失败: ', error);
    });
  } else {
    bgMusic.value.pause();
  }
});

onMounted(() => {
  window.addEventListener('keydown', handleKeyPress);
  window.addEventListener('keyup', handleKeyRelease);
});

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleKeyPress);
  window.removeEventListener('keyup', handleKeyRelease);
});
</script>

<style scoped>
.container {
  position: relative;
  background-color: #1d1a1a;
  color: white;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 10px;
}

.muyu-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.muyu-group {
  position: relative;
}

.counter {
  font-size: 110px;
  margin-bottom: 5px;
}

.description {
  font-size: 40px;
  color: gray;
  margin-bottom: 20px;
}

.muyu-icon {
  width: 150px;
  cursor: pointer;
  transition: transform 0.1s ease;
  margin-bottom: 20px;
}

.hit-animation {
  transform: scale(1.3);
}

.stick-icon {
  position: absolute;
  top: -20px;
  left: -40px;
  width: 120px;
  transition: transform 0.3s ease;
}

.stick-hit-animation {
  transform: rotate(15deg);
}

.bonus-text {
  display: inline-block;
  position: absolute;
  top: -35px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 24px;
  color: white;
  text-shadow: 2px 2px 8px rgba(255, 255, 255, 0.3);
  animation: textFade 1.2s ease-out forwards;
  white-space: nowrap;
}

@keyframes textFade {
  0% {
    opacity: 1;
    transform: translateY(0);
  }
  100% {
    opacity: 0;
    transform: translateY(-25px);
  }
}

.click-text {
  font-size: 16px;
  color: gray;
  margin-top: 10px;
  text-align: center;
}

.switch-options-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
  margin-top: 20px;
  transform: scale(1.2);
}

.option {
  font-size: 16px;
  color: #ccc;
}

.underline {
  text-decoration: underline;
}

.el-radio-group {

  display: flex;
  flex-direction: column;
  gap: 15px; /* 使选项之间有合适的间距 */
  align-items: flex-start; /* 让选项整齐排列 */
}




</style>
