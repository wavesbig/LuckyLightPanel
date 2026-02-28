<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useConfigStore } from '@/stores/config'
import { Search, ChevronDown, Check, Sparkles, ExternalLink, Zap } from 'lucide-vue-next'
import type { SearchEngine } from '@/types'

// 导入本地图标
import iconBing from '@/assets/icons/bing.png'
import iconGoogle from '@/assets/icons/google.ico'
import iconBaidu from '@/assets/icons/baidu.ico'
import iconSogou from '@/assets/icons/sogou.ico'
import icon360 from '@/assets/icons/so.ico'
import iconGithub from '@/assets/icons/github.ico'

const configStore = useConfigStore()

// 搜索关键词
const searchQuery = ref('')

// 搜索引擎下拉菜单状态
const engineDropdownOpen = ref(false)

// 预设搜索引擎列表
const SEARCH_ENGINES: SearchEngine[] = [
  { id: 'bing', name: 'Bing', url: 'https://www.bing.com/search?q={query}', icon: iconBing },
  { id: 'google', name: 'Google', url: 'https://www.google.com/search?q={query}', icon: iconGoogle },
  { id: 'baidu', name: '百度', url: 'https://www.baidu.com/s?wd={query}', icon: iconBaidu },
  { id: 'sogou', name: '搜狗', url: 'https://www.sogou.com/web?query={query}', icon: iconSogou },
  { id: '360', name: '360搜索', url: 'https://www.so.com/s?q={query}', icon: icon360 },
  { id: 'github', name: 'GitHub', url: 'https://github.com/search?q={query}', icon: iconGithub }
]

// 当前搜索引擎
const currentEngine = computed(() => {
  const engineId = configStore.searchEngine
  if (engineId === 'custom') {
    return { id: 'custom', name: '自定义', url: configStore.config.customSearchUrl || '', icon: '' }
  }
  return SEARCH_ENGINES.find(e => e.id === engineId) || SEARCH_ENGINES[0]
})

// 执行搜索
function doSearch() {
  const query = searchQuery.value.trim()
  if (!query) return
  
  let searchUrl = currentEngine.value.url
  if (!searchUrl) return
  
  // 替换查询占位符
  searchUrl = searchUrl.replace('{query}', encodeURIComponent(query))
  
  // 在新标签页打开
  window.open(searchUrl, '_blank')
}

// 处理回车搜索
function handleKeydown(event: KeyboardEvent) {
  if (event.key === 'Enter') {
    doSearch()
  }
}

// 选择搜索引擎
function selectEngine(engineId: string) {
  configStore.setSearchEngine(engineId)
  engineDropdownOpen.value = false
}

// 切换下拉菜单
function toggleEngineDropdown() {
  engineDropdownOpen.value = !engineDropdownOpen.value
}

// 点击外部关闭下拉菜单
function closeEngineDropdown(event: MouseEvent) {
  const target = event.target as HTMLElement
  if (!target.closest('.engine-dropdown-wrapper')) {
    engineDropdownOpen.value = false
  }
}

onMounted(() => {
  document.addEventListener('click', closeEngineDropdown)
})

onUnmounted(() => {
  document.removeEventListener('click', closeEngineDropdown)
})
</script>

<template>
  <div class="search-bar-container">
    <!-- 光晕背景层 -->
    <div class="search-glow" />
    
    <div class="search-bar glass-search">
      <!-- 顶部高光线 -->
      <div class="search-highlight" />
      
      <!-- 搜索引擎选择器 -->
      <div class="engine-dropdown-wrapper">
        <button 
          class="engine-selector"
          :class="{ active: engineDropdownOpen }"
          @click.stop="toggleEngineDropdown"
          :title="currentEngine.name"
        >
          <img v-if="currentEngine.icon" :src="currentEngine.icon" :alt="currentEngine.name" class="engine-icon" />
          <Zap v-else class="engine-icon-svg" :size="16" />
          <span class="engine-name">{{ currentEngine.name }}</span>
          <ChevronDown class="dropdown-arrow" :class="{ open: engineDropdownOpen }" />
        </button>
        
        <!-- 下拉菜单 -->
        <Transition
          enter-active-class="dropdown-enter"
          enter-from-class="dropdown-enter-from"
          leave-active-class="dropdown-leave"
          leave-to-class="dropdown-leave-to"
        >
          <div v-if="engineDropdownOpen" class="engine-dropdown">
            <div class="dropdown-header">
              <Sparkles class="header-icon" />
              <span>搜索引擎</span>
            </div>
            <div class="dropdown-options">
              <button
                v-for="engine in SEARCH_ENGINES"
                :key="engine.id"
                class="engine-option"
                :class="{ active: configStore.searchEngine === engine.id }"
                @click="selectEngine(engine.id)"
              >
                <img :src="engine.icon" :alt="engine.name" class="option-icon" />
                <span class="option-label">{{ engine.name }}</span>
                <Check v-if="configStore.searchEngine === engine.id" class="option-check" />
              </button>
              <!-- 自定义搜索引擎选项（仅当自定义URL不为空时显示） -->
              <button
                v-if="configStore.config.customSearchUrl"
                class="engine-option"
                :class="{ active: configStore.searchEngine === 'custom' }"
                @click="selectEngine('custom')"
              >
                <Zap class="option-icon-svg" :size="16" />
                <span class="option-label">自定义</span>
                <Check v-if="configStore.searchEngine === 'custom'" class="option-check" />
              </button>
            </div>
          </div>
        </Transition>
      </div>
      
      <!-- 分隔线 -->
      <div class="input-divider" />
      
      <!-- 搜索输入框 -->
      <input
        v-model="searchQuery"
        type="text"
        class="search-input"
        :placeholder="`搜索你想要的内容...`"
        @keydown="handleKeydown"
      />
      
      <!-- 搜索按钮 -->
      <button class="search-btn" @click="doSearch" title="搜索">
        <div class="btn-bg" />
        <Search class="search-icon" />
        <ExternalLink class="external-icon" />
      </button>
    </div>
  </div>
</template>

<style scoped>
/* ============================================
   搜索栏容器 - Pro Max 级设计
   ============================================ */
.search-bar-container {
  position: relative;
  width: 100%;
  max-width: 720px;
  margin: 0.75rem auto 0.5rem;
  padding: 0 1rem;
  z-index: 100;
}

/* 光晕背景层 */
.search-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 80%;
  height: 200%;
  background: radial-gradient(
    ellipse at center,
    hsl(var(--neon-cyan) / 0.08) 0%,
    transparent 70%
  );
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.4s ease;
  filter: blur(20px);
}

.search-bar-container:hover .search-glow,
.search-bar-container:focus-within .search-glow {
  opacity: 1;
}

.search-bar {
  position: relative;
  display: flex;
  align-items: center;
  height: 3rem; /* 从 3.5rem (56px) 缩小到 3rem (48px) */
  border-radius: 1.5rem; /* 配合缩小的高度 */
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1);
}

/* 顶部高光线 */
.search-highlight {
  position: absolute;
  top: 0;
  left: 10%;
  right: 10%;
  height: 1px;
  background: linear-gradient(
    90deg,
    transparent,
    hsl(var(--text-primary) / 0.1) 30%,
    hsl(var(--text-primary) / 0.15) 50%,
    hsl(var(--text-primary) / 0.1) 70%,
    transparent
  );
  pointer-events: none;
}

/* 毛玻璃搜索栏核心质感 - 匹配侧边栏 */
.glass-search {
  background: hsl(var(--glass-bg));
  backdrop-filter: blur(var(--glass-blur));
  -webkit-backdrop-filter: blur(var(--glass-blur));
  border: 1px solid hsl(var(--glass-border));
  box-shadow: var(--shadow-xl), var(--glass-border-glow);
}

.glass-search:hover {
  background: hsl(var(--glass-bg));
  border-color: hsl(var(--primary) / 0.4);
  transform: translateY(-1px);
  box-shadow: 
    var(--shadow-xl), 
    0 8px 24px -6px hsl(var(--primary) / 0.15);
}

.glass-search:focus-within {
  background: hsl(var(--glass-bg));
  border-color: hsl(var(--primary) / 0.6);
  box-shadow: 
    var(--shadow-xl),
    0 0 0 2px hsl(var(--primary) / 0.2),
    0 16px 32px -8px hsl(var(--primary) / 0.15);
}

/* ============================================
   搜索引擎选择器
   ============================================ */
.engine-dropdown-wrapper {
  position: relative;
  flex-shrink: 0;
  height: 100%;
  display: flex;
  align-items: center;
}

.engine-selector {
  display: flex;
  align-items: center;
  gap: 0.375rem;
  padding: 0 0.625rem 0 0.875rem; /* 左右微微不对称调整视觉重心 */
  height: calc(100% - 1rem); /* 上下留有缝隙，构成子胶囊 */
  margin-left: 0.5rem;
  background: transparent;
  border: none;
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  color: hsl(var(--text-secondary));
  border-radius: 1.5rem; /* 完整的全圆角 */
}

.engine-selector:hover {
  background: hsl(var(--primary) / 0.08); /* 极淡的主题色背景 */
  color: hsl(var(--text-primary));
}

.engine-selector.active {
  background: hsl(var(--primary) / 0.12); /* 选中时的深一点主题色 */
  color: hsl(var(--primary));
  box-shadow: inset 0 0 0 1px hsl(var(--primary) / 0.1);
}

.engine-icon {
  width: 1.25rem;
  height: 1.25rem;
  object-fit: contain;
  flex-shrink: 0;
}

.engine-icon-svg {
  width: 1.25rem;
  height: 1.25rem;
  flex-shrink: 0;
  color: hsl(var(--neon-cyan));
}

.engine-name {
  font-size: 0.8125rem;
  font-weight: 500;
  letter-spacing: 0.01em;
}

.dropdown-arrow {
  width: 1rem;
  height: 1rem;
  color: hsl(var(--text-muted));
  transition: transform 0.25s ease;
  margin-left: 0.125rem;
}

.dropdown-arrow.open {
  transform: rotate(180deg);
  color: hsl(var(--neon-cyan));
}

/* ============================================
   搜索引擎下拉菜单
   ============================================ */
.engine-dropdown {
  position: absolute;
  top: calc(100% + 1rem); /* 让弹出的菜单离胶囊按钮远一些 */
  left: 0;
  min-width: 220px; /* 增加最小宽度 */
  border-radius: var(--radius-xl);
  background: hsl(var(--bg-elevated) / 0.85); /* 提升通透感 */
  backdrop-filter: blur(var(--glass-blur, 30px)) saturate(1.8);
  -webkit-backdrop-filter: blur(var(--glass-blur, 30px)) saturate(1.8);
  border: 1px solid hsl(var(--glass-border) / 0.8);
  box-shadow: 
    var(--shadow-xl), 
    0 0 0 1px hsl(var(--neon-cyan) / 0.05),
    inset 0 1px 0 hsl(0 0% 100% / 0.1);
  z-index: 9999;
  overflow: hidden;
  transform-origin: top left;
}

.dropdown-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 1rem 1.25rem;
  font-size: 0.75rem;
  font-weight: 700;
  color: hsl(var(--text-secondary));
  border-bottom: 1px solid hsl(var(--border-subtle) / 0.4);
  text-transform: uppercase;
  letter-spacing: 0.08em;
  background: transparent;
}

.header-icon {
  width: 0.875rem;
  height: 0.875rem;
  color: hsl(var(--neon-cyan));
  animation: sparkle 2s ease-in-out infinite;
}

@keyframes sparkle {
  0%, 100% { opacity: 0.7; transform: scale(1); }
  50% { opacity: 1; transform: scale(1.1); }
}

.dropdown-options {
  padding: 0.6rem;
}

.engine-option {
  display: flex;
  align-items: center;
  gap: 0.875rem;
  width: 100%;
  padding: 0.75rem 1rem;
  border-radius: var(--radius-lg);
  background: transparent;
  border: none;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  color: hsl(var(--text-secondary));
}

.engine-option:hover {
  background: hsl(var(--primary) / 0.08); /* 使用主题色取代之前的灰色 */
  color: hsl(var(--text-primary));
  transform: translateX(4px); /* 放大反馈 */
}

.engine-option.active {
  background: hsl(var(--primary) / 0.12);
  color: hsl(var(--primary));
  font-weight: 600;
}

.option-icon {
  width: 1.125rem;
  height: 1.125rem;
  object-fit: contain;
  flex-shrink: 0;
}

.option-icon-svg {
  width: 1.125rem;
  height: 1.125rem;
  flex-shrink: 0;
  color: hsl(var(--neon-cyan));
}

.option-label {
  flex: 1;
  font-size: 0.875rem;
  font-weight: 500;
  text-align: left;
}

.option-check {
  width: 1rem;
  height: 1rem;
  color: hsl(var(--neon-cyan));
}

/* 下拉菜单动画 */
.dropdown-enter {
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
}

.dropdown-enter-from {
  opacity: 0;
  transform: translateY(-12px) scale(0.95);
}

.dropdown-leave {
  transition: all 0.2s ease-in;
}

.dropdown-leave-to {
  opacity: 0;
  transform: translateY(-8px) scale(0.95);
}

/* ============================================
   输入分隔线
   ============================================ */
.input-divider {
  width: 1px;
  height: 1.25rem;
  background: hsl(var(--border-subtle) / 0.5); /* 摈弃复杂的渐变 */
  margin: 0 0.5rem 0 0.25rem;
}

/* ============================================
   搜索输入框
   ============================================ */
.search-input {
  flex: 1;
  height: 100%;
  padding: 0 1.25rem;
  background: transparent;
  border: none;
  outline: none;
  font-size: 1rem;
  font-weight: 400;
  color: hsl(var(--text-primary));
  letter-spacing: 0.01em;
}

.search-input::placeholder {
  color: hsl(var(--text-muted));
  font-weight: 400;
}

/* ============================================
   搜索按钮 - 简约圆形设计
   ============================================ */
.search-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 2.25rem; /* 正圆形 36px */
  height: 2.25rem;
  margin: 0.375rem; /* 与右边缘保持同等圆角间距 */
  border-radius: 50%;
  background: hsl(var(--primary) / 0.15); /* 提高一点默认背景透明度 */
  color: hsl(var(--primary));
  border: 1px solid hsl(var(--primary) / 0.2); /* 增加一个同色系极细边框以凸出轮廓 */
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  overflow: hidden;
  flex-shrink: 0;
}

.btn-bg {
  position: absolute;
  inset: 0;
  background: hsl(var(--primary));
  opacity: 0;
  transition: opacity 0.3s ease;
}

.search-btn:hover .btn-bg {
  opacity: 1;
}

.search-btn:hover {
  color: white; /* 悬停时图标反白 */
  transform: scale(1.05);
  box-shadow: 0 4px 12px -2px hsl(var(--primary) / 0.4);
}

.search-btn:active {
  transform: scale(0.95);
}

.search-icon,
.external-icon {
  width: 1.25rem;
  height: 1.25rem;
  color: hsl(var(--primary)); /* 恢复为主题色，而不是强制白色 */
  position: absolute;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 1;
}

.search-btn:hover .search-icon,
.search-btn:hover .external-icon {
  color: white; /* 悬停时才反白 */
}

.search-icon {
  opacity: 1;
  transform: translateY(0) scale(1);
}

.external-icon {
  opacity: 0;
  transform: translateY(10px) scale(0.8);
}

.search-btn:hover .search-icon {
  opacity: 0;
  transform: translateY(-10px) scale(0.8);
}

.search-btn:hover .external-icon {
  opacity: 1;
  transform: translateY(0) scale(1);
}

/* ============================================
   响应式调整
   ============================================ */
@media (min-width: 640px) {
  .search-bar-container {
    padding: 0 1.5rem;
  }
  
  .search-bar {
    height: 3.25rem; /* PC端稍微加大到 52px */
    border-radius: 1.625rem;
  }
  
  .search-btn {
    width: 2.5rem; /* 40px */
    height: 2.5rem;
  }
}

@media (max-width: 640px) {
  .engine-name {
    display: none;
  }
  
  .engine-selector {
    padding: 0 0.875rem;
    border-right: none;
  }
}

@media (max-width: 480px) {
  .search-bar-container {
    padding: 0 0.75rem;
  }
  
  .search-bar {
    height: 3rem;
    border-radius: 1.5rem;
  }
  
  .engine-selector {
    padding: 0 0.75rem;
  }
  
  .search-input {
    font-size: 0.9375rem;
    padding: 0 0.875rem;
  }
  
  .search-btn {
    width: 2.25rem;
    height: 2.25rem;
  }
  
  .search-icon,
  .external-icon {
    width: 1.125rem;
    height: 1.125rem;
  }
}
</style>
