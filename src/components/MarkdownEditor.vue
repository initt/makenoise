<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, watch } from 'vue';
import { marked } from 'marked';
import FluidAnimation from './FluidAnimation.vue';
import { Download, Copy, Trash2, Sun, Moon, Github } from 'lucide-vue-next';

// --- Constants ---
const STORAGE_KEY = 'markdown-editor-content';

// --- State ---
const theme = ref('dark'); // 'dark' or 'light'
const markdown = ref(''); // Start with empty and load from storage
const editorWidth = ref(50);
const isDragging = ref(false);
const copyButtonText = ref('Copy');
const isMobile = ref(false);

// --- Computed ---
const renderedHtml = computed(() => {
  return marked(markdown.value);
});

const isMarkdownEmpty = computed(() => {
  return markdown.value.trim() === '';
});

const editorStyle = computed(() => ({
  width: isMobile.value ? '100%' : `calc(${editorWidth.value}% - 1px)`
}));

const previewStyle = computed(() => ({
  width: isMobile.value ? '100%' : `calc(${100 - editorWidth.value}% - 1px)`
}));

const themeIcon = computed(() => (theme.value === 'dark' ? Sun : Moon));

// --- Methods ---
const toggleTheme = () => {
  theme.value = theme.value === 'dark' ? 'light' : 'dark';
};

const downloadContent = () => {
  const blob = new Blob([markdown.value], { type: 'text/markdown;charset=utf-8' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'markdown.md';
  a.click();
  URL.revokeObjectURL(url);
};

const copyMarkdown = () => {
  navigator.clipboard.writeText(markdown.value).then(() => {
    copyButtonText.value = 'Copied!';
    setTimeout(() => {
      copyButtonText.value = 'Copy';
    }, 2000);
  });
};

const clearContent = () => {
  if (window.confirm('Are you sure you want to clear all content? This cannot be undone.')) {
    markdown.value = '';
  }
};

const startDrag = () => {
  if (isMobile.value) return; // Disable drag on mobile
  isDragging.value = true;
  document.body.style.cursor = 'col-resize';
  document.body.style.userSelect = 'none';
};

const stopDrag = () => {
  isDragging.value = false;
  document.body.style.cursor = '';
  document.body.style.userSelect = '';
};

const onDrag = (event: MouseEvent) => {
  if (isDragging.value && !isMobile.value) {
    const newWidth = (event.clientX / window.innerWidth) * 100;
    if (newWidth > 15 && newWidth < 85) {
      editorWidth.value = newWidth;
    }
  }
};

const checkMobile = () => {
  isMobile.value = window.innerWidth <= 768;
};

// --- Watchers ---
watch(markdown, (newContent) => {
  localStorage.setItem(STORAGE_KEY, newContent);
});

// --- Lifecycle Hooks ---
onMounted(() => {
  const savedContent = localStorage.getItem(STORAGE_KEY);
  if (savedContent) {
    markdown.value = savedContent;
  } else {
    markdown.value = '';
  }

  checkMobile();
  window.addEventListener('mousemove', onDrag);
  window.addEventListener('mouseup', stopDrag);
  window.addEventListener('resize', checkMobile);
});

onUnmounted(() => {
  window.removeEventListener('mousemove', onDrag);
  window.removeEventListener('mouseup', stopDrag);
  window.removeEventListener('resize', checkMobile);
});

</script>

<template>
  <div class="editor-container" :class="{ 'light-theme': theme === 'light' }">
    <div class="header">
      <div class="logo">
        <img src="/makenoise.svg" alt="MakeNoise" class="logo-image" />
        <span class="logo-text">MakeNoise</span>
      </div>
      <div class="header-controls">
        <button @click="toggleTheme" class="theme-toggle">
          <component :is="themeIcon" class="theme-icon" />
        </button>
        <a href="https://github.com/initt/makenoise" target="_blank" rel="noopener noreferrer" class="github-link">
          <Github class="github-icon" />
        </a>
      </div>
    </div>

    <div class="main-content">
      <div class="editor" :style="editorStyle">
        <div class="action-bar">
          <button @click="downloadContent">
            <Download class="button-icon" />
            <span>Download</span>
          </button>
          <button @click="copyMarkdown">
            <Copy class="button-icon" />
            <span>{{ copyButtonText }}</span>
          </button>
          <button @click="clearContent" class="btn-danger">
            <Trash2 class="button-icon" />
            <span>Clear</span>
          </button>
        </div>
        <textarea v-model="markdown"></textarea>
      </div>
      <div class="divider" @mousedown.prevent="startDrag"></div>
      <div class="preview" :style="previewStyle">
        <FluidAnimation v-if="isMarkdownEmpty" style="width: 100%; height: 100%;" />
        <div v-else v-html="renderedHtml" class="preview-content" style="width: 100%; height: 100%;"></div>
      </div>
    </div>

    <footer class="footer">
      <div class="footer-content">
        <div class="footer-left">
          <span class="footer-text">{{ markdown.length }} characters</span>
        </div>
        <div class="footer-center">
          <span class="footer-text">2025 MakeNoise Markdown Editor</span>
        </div>
        <div class="footer-right">
          <span class="footer-text">100% VIBE CODING</span>
        </div>
      </div>
    </footer>
  </div>
</template>

<style scoped>
.editor-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 100vw;
  font-size: 15px;
  background-color: var(--background);
  color: var(--text-primary);
  transition: background-color 0.3s, color 0.3s;
  overflow: hidden;
  position: relative;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  border-bottom: 1px solid var(--border-subtle);
  flex-shrink: 0;
  background-color: var(--background);
  z-index: 10;
}

.logo {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.logo-image {
  width: 32px;
  height: 32px;
  filter: var(--logo-filter);
}

.logo-text {
  font-size: 1.2rem;
  font-weight: 600;
  color: var(--text-primary);
}

.header-controls {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.main-content {
  display: flex;
  flex: 1;
  min-height: 0;
}

/* Mobile responsive layout */
@media (max-width: 768px) {
  .header {
    padding: 0.5rem 1rem;
  }

  .logo-image {
    width: 28px;
    height: 28px;
  }

  .logo-text {
    font-size: 1rem;
  }

  .main-content {
    flex-direction: column;
    height: 100vh;
    padding-bottom: 50px; /* Footer height */
    box-sizing: border-box;
  }

  .editor,
  .preview {
    flex: 1;
    min-height: 0;
    display: flex;
    flex-direction: column;
    border: 1px solid var(--border-subtle);
    border-radius: 8px;
    margin: 4px;
  }

  .divider {
    display: none;
  }

  .theme-toggle {
    top: 0.5rem;
    right: 3rem;
    width: 36px;
    height: 36px;
    min-height: 36px;
    box-sizing: border-box;
  }

  .github-link {
    top: 0.5rem;
    right: 0.5rem;
    width: 36px;
    height: 36px;
    min-height: 36px;
    box-sizing: border-box;
  }

  .theme-icon,
  .github-icon {
    width: 18px;
    height: 18px;
  }

  .action-bar {
    padding: 0.5rem 1rem;
    gap: 0.3rem;
    flex-shrink: 0;
  }

  .action-bar button {
    padding: 0.3rem 0.6rem;
    font-size: 12px;
    gap: 0.3rem;
    height: 44px;
    min-height: 44px;
    box-sizing: border-box;
  }

  /* Hide button text on mobile, show only icons */
  .action-bar button span {
    display: none;
  }

  .button-icon {
    width: 14px;
    height: 14px;
  }

  .editor textarea {
    padding: 1rem;
    font-size: 16px; /* Prevent zoom on iOS */
    flex: 1;
    min-height: 0;
    border: none;
    resize: none;
    background-color: transparent;
    color: var(--text-primary);
    font-family: 'Menlo', 'Consolas', monospace;
    line-height: 1.6;
    outline: none;
    box-sizing: border-box;
  }

  .preview {
    padding: 1rem;
    overflow-y: auto;
    box-sizing: border-box;
  }

  .preview-content {
    width: 100%;
    height: 100%;
    overflow-y: auto;
  }

  .footer {
    padding: 0.5rem 1rem;
    flex-shrink: 0;
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    height: 50px; /* Fixed height */
    z-index: 100;
    background-color: var(--background);
    border-top: 1px solid var(--border-subtle);
    box-sizing: border-box;
    display: flex;
    align-items: center;
  }

  .footer-content {
    flex-direction: row;
    justify-content: space-between;
    width: 100%;
  }

  .footer-left,
  .footer-center,
  .footer-right {
    text-align: center;
  }

  .footer-text {
    font-size: 11px;
  }

  /* Hide footer-left and footer-right on mobile, show only copyright */
  .footer-left,
  .footer-right {
    display: none;
  }

  .footer-center {
    text-align: center;
    flex: 1;
  }
}

.theme-toggle {
    background: transparent;
    border: 1px solid var(--border-subtle);
    color: var(--text-secondary);
    border-radius: 50%;
    width: 40px;
    height: 40px;
    min-height: 40px;
    box-sizing: border-box;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    transition: background-color 0.2s, color 0.2s;
}

.github-link {
    background: transparent;
    border: 1px solid var(--border-subtle);
    color: var(--text-secondary);
    border-radius: 50%;
    width: 40px;
    height: 40px;
    min-height: 40px;
    box-sizing: border-box;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    text-decoration: none;
    transition: background-color 0.2s, color 0.2s;
}

.github-icon {
    width: 20px;
    height: 20px;
}

.github-link:hover {
    background-color: var(--border-subtle);
    color: var(--text-primary);
}

.theme-icon {
    width: 20px;
    height: 20px;
}
.theme-toggle:hover {
    background-color: var(--border-subtle);
}

.editor {
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 0; /* Remove padding to make action-bar and textarea flush */
  min-height: 0; /* Allow flex shrinking */
  background-color: rgba(255, 255, 255, 0.01); /* Slightly lighter background */
  border-right: 1px solid var(--border-subtle);
}

.preview {
  height: 100%;
  padding: 2rem;
  box-sizing: border-box;
  overflow-y: auto;
  overflow-x: hidden;
  min-height: 0; /* Allow flex shrinking */
  border-left: 1px solid var(--border-subtle);
}

.action-bar {
  display: flex;
  padding: 0.5rem 2rem;
  border-bottom: 1px solid var(--border-subtle);
  flex-shrink: 0;
  gap: 0.5rem;
}

.action-bar button {
  background-color: transparent;
  border: 1px solid var(--border-subtle);
  color: var(--text-secondary);
  padding: 0.4rem 0.8rem;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  transition: background-color 0.2s, color 0.2s;
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

.button-icon {
  width: 16px;
  height: 16px;
}

.action-bar button:hover {
  background-color: var(--border-subtle);
  color: var(--text-primary);
}

.action-bar .btn-danger:hover {
  background-color: #4d1b1b;
  border-color: #842020;
  color: #ffb1b1;
}

.editor textarea {
  width: 100%;
  height: 100%;
  border: none;
  resize: none;
  background-color: transparent;
  color: var(--text-primary);
  font-family: 'Menlo', 'Consolas', monospace;
  font-size: 14px;
  line-height: 1.6;
  outline: none;
  padding: 2rem; /* Add padding back to the textarea itself */
  box-sizing: border-box;
  overflow-y: auto;
  flex: 1;
  min-height: 0;
}

.divider {
  width: 1px;
  height: 100%;
  cursor: col-resize;
  background-color: var(--border-subtle);
  flex-shrink: 0;
  transition: background-color 0.3s;
}

.divider:hover {
    background-color: #444;
}

/* ... preview styles ... */
:deep(.preview h1), :deep(.preview h2), :deep(.preview h3) {
  color: var(--text-primary);
  font-weight: 600;
  margin-bottom: 0.5em;
  border-bottom: 1px solid var(--border-subtle);
  padding-bottom: 0.3em;
  transition: color 0.3s, border-color 0.3s;
}
:deep(.preview h1) { font-size: 2em; }
:deep(.preview h2) { font-size: 1.5em; }
:deep(.preview h3) { font-size: 1.2em; }
:deep(.preview p) { margin-top: 0; margin-bottom: 1.2em; }
:deep(.preview a) { color: #5e9eff; text-decoration: none; }
:deep(.preview a:hover) { text-decoration: underline; }
:deep(.preview code) {
  background-color: rgba(110, 118, 129, 0.2);
  padding: 0.2em 0.4em;
  border-radius: 6px;
  font-family: 'Menlo', 'Consolas', monospace;
  font-size: 0.9em;
  color: var(--text-primary);
}
:deep(.preview pre) {
  background-color: rgba(110, 118, 129, 0.1);
  padding: 1em;
  border-radius: 8px;
  overflow-x: auto;
}
:deep(.preview pre code) { background-color: transparent; padding: 0; }
:deep(.preview ul), :deep(.preview ol) { padding-left: 1.5em; }
:deep(.preview blockquote) {
  margin-left: 0;
  padding-left: 1em;
  border-left: 3px solid var(--border-subtle);
  color: var(--text-secondary);
  transition: color 0.3s, border-color 0.3s;
}
:deep(.preview hr) {
  border: none;
  height: 1px;
  background-color: var(--border-subtle);
  margin: 2em 0;
  transition: background-color 0.3s;
}

/* Mobile responsive preview styles */
@media (max-width: 768px) {
  :deep(.preview h1) { font-size: 1.6em; }
  :deep(.preview h2) { font-size: 1.3em; }
  :deep(.preview h3) { font-size: 1.1em; }
  :deep(.preview p) { margin-bottom: 1em; }
  :deep(.preview pre) { padding: 0.8em; }
  :deep(.preview ul), :deep(.preview ol) { padding-left: 1.2em; }
  :deep(.preview blockquote) { padding-left: 0.8em; }
}

.footer {
  background-color: var(--border-subtle);
  border-top: 1px solid var(--border-subtle);
  padding: 0.75rem 2rem;
  flex-shrink: 0;
}

.footer-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 100%;
}

.footer-left,
.footer-center,
.footer-right {
  flex: 1;
}

.footer-center {
  text-align: center;
}

.footer-right {
  text-align: right;
}

.footer-text {
  font-size: 12px;
  color: var(--text-secondary);
  font-weight: 500;
}
</style>

