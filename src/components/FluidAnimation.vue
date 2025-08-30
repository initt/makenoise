<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { animate } from 'animejs';

const fluidContainer = ref<HTMLElement>();

// 複雑な流体パターンを生成する関数
const generateComplexPattern = (frame: number) => {
  const lines = [];
  
  // プレビューエリア内に収まる行数に調整
  for (let i = 0; i < 35; i++) {
    let line = '';
    // プレビューエリア内に収まる文字数に調整（横スクロール防止）
    for (let j = 0; j < 130; j++) {
      // 複数のノイズ関数を組み合わせて有機的なパターンを生成
      const noise1 = Math.sin((j + frame * 0.5) * 0.1 + i * 0.2) * 0.5 + 0.5;
      const noise2 = Math.cos((j + frame * 0.3) * 0.15 + i * 0.1) * 0.5 + 0.5;
      const noise3 = Math.sin((i + frame * 0.2) * 0.3 + j * 0.05) * 0.5 + 0.5;
      
      // 複数のノイズを組み合わせて密度を制御
      const density = (noise1 + noise2 + noise3) / 3;
      
      // 密度に基づいて文字を選択
      let char;
      if (density > 0.8) {
        // 高密度エリア: より複雑な文字
        const highDensityChars = '*##=+';
        char = highDensityChars[Math.floor(Math.random() * highDensityChars.length)];
      } else if (density > 0.6) {
        // 中密度エリア: 中程度の文字
        const mediumDensityChars = '~_|/\\^';
        char = mediumDensityChars[Math.floor(Math.random() * mediumDensityChars.length)];
      } else if (density > 0.4) {
        // 低密度エリア: シンプルな文字
        const lowDensityChars = '()[]{}<>!?@$&%';
        char = lowDensityChars[Math.floor(Math.random() * lowDensityChars.length)];
      } else {
        // 非常に低密度エリア: 最小限の文字
        const minimalChars = ':.';
        char = minimalChars[Math.floor(Math.random() * minimalChars.length)];
      }
      
      // 時々空白を挿入してテクスチャを調整
      if (Math.random() < 0.1) {
        char = ' ';
      }
      
      line += char;
    }
    lines.push(line);
  }
  return lines.join('\n');
};

// 流体力学のASCIIパターン（複雑パターン）
const fluidFrames = [
  generateComplexPattern(0),
  generateComplexPattern(1),
  generateComplexPattern(2),
  generateComplexPattern(3)
];

const currentFrame = ref(0);
let animationId: ReturnType<typeof setTimeout>;

const startAnimation = () => {
  const animateFluid = () => {
    // 新しい複雑パターンを生成
    fluidFrames[currentFrame.value] = generateComplexPattern(currentFrame.value);
    currentFrame.value = (currentFrame.value + 1) % fluidFrames.length;
    animationId = setTimeout(animateFluid, 150);
  };
  animateFluid();
};

const stopAnimation = () => {
  if (animationId) {
    clearTimeout(animationId);
  }
};

onMounted(() => {
  startAnimation();
  
  // anime.jsでコンテナをフェードイン
  if (fluidContainer.value) {
    animate(fluidContainer.value, {
      opacity: [0, 1],
      scale: [0.95, 1],
      duration: 1800,
      ease: 'easeOutCubic',
    });
  }
});

onUnmounted(() => {
  stopAnimation();
});
</script>

<template>
  <div ref="fluidContainer" class="fluid-animation">
    <pre class="ascii-art">{{ fluidFrames[currentFrame] }}</pre>
    <div class="center-text">[make noise, make code]</div>
  </div>
</template>

<style scoped>
.fluid-animation {
  position: relative;
  height: 100%;
  opacity: 0;
  transform: scale(0.95);
  padding: 1rem;
  color: var(--text-secondary);
  overflow: hidden;
  width: 100%;
  box-sizing: border-box;
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.ascii-art {
  font-family: 'Courier New', monospace;
  font-size: 0.5rem;
  line-height: 0.8;
  color: var(--text-secondary);
  white-space: pre;
  text-align: center;
  opacity: 0.8;
  letter-spacing: 0.1px;
  margin: 0;
  padding: 0;
  max-width: 100%;
  overflow-wrap: break-word;
  word-wrap: break-word;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Mobile responsive adjustments */
@media (max-width: 768px) {
  .ascii-art {
    font-size: 0.4rem;
    line-height: 0.7;
  }
  
  .center-text {
    font-size: 1rem;
    letter-spacing: 2px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: 95vw;
  }
}

/* Small mobile devices */
@media (max-width: 480px) {
  .center-text {
    font-size: 0.8rem;
    letter-spacing: 1px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: 98vw;
  }
}

/* Extra small mobile devices */
@media (max-width: 360px) {
  .center-text {
    font-size: 0.6rem;
    letter-spacing: 0.3px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: 100vw;
  }
}

.center-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-family: 'Inter', sans-serif;
  font-size: 1.2rem;
  font-weight: 100;
  color: var(--text-primary);
  z-index: 2;
  letter-spacing: 3px;
  pointer-events: none;
  text-shadow: 0 0 15px rgba(0, 0, 0, 0.3);
  transition: color 0.3s ease, text-shadow 0.3s ease;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 90vw;
}

/* ライトモードでのテキストシャドウ調整 */
.light-theme .center-text {
  text-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
}
</style>
