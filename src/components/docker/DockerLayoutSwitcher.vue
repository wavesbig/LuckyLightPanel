<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed, nextTick } from "vue";
import { useConfigStore } from "@/stores/config";
import type { LayoutMode } from "@/types";

const configStore = useConfigStore();

const isOpen = ref(false);
const dropdownRef = ref<HTMLElement | null>(null);
const dropdownMenuRef = ref<HTMLElement | null>(null);
const alignRight = ref(false);

// Docker 布局选项配置 - 四种布局
const layoutOptions: {
  value: LayoutMode;
  label: string;
  icon: string;
  desc: string;
}[] = [
  { value: "list", label: "列表", icon: "list", desc: "横向展示" },
  { value: "normal", label: "详情", icon: "detail", desc: "完整统计" },
  { value: "compact", label: "紧凑", icon: "compact", desc: "精简显示" },
  { value: "minimal", label: "极简", icon: "minimal", desc: "状态预览" },
];

// 当前选中的布局
const currentLayout = computed(() => configStore.dockerLayout);

// 获取当前布局的显示名称
const displayLabel = computed(() => {
  const option = layoutOptions.find((o) => o.value === currentLayout.value);
  return option?.label || "详情";
});

// 检测下拉菜单是否超出右边界
function checkDropdownPosition() {
  nextTick(() => {
    if (dropdownMenuRef.value && dropdownRef.value) {
      const dropdown = dropdownMenuRef.value;
      const wrapper = dropdownRef.value;
      const wrapperRect = wrapper.getBoundingClientRect();
      const dropdownWidth = dropdown.offsetWidth;
      const viewportWidth = window.innerWidth;

      // 计算居中时下拉菜单右边界位置
      const centerRight =
        wrapperRect.left + wrapperRect.width / 2 + dropdownWidth / 2;

      // 如果超出视口右边界，则右对齐
      alignRight.value = centerRight > viewportWidth - 16;
    }
  });
}

// 切换下拉菜单
function toggleDropdown() {
  isOpen.value = !isOpen.value;
  if (isOpen.value) {
    checkDropdownPosition();
  }
}

// 选择布局
function selectLayout(layout: LayoutMode) {
  configStore.setDockerLayout(layout);
  isOpen.value = false;
}

// 点击外部关闭
function handleClickOutside(event: MouseEvent) {
  if (dropdownRef.value && !dropdownRef.value.contains(event.target as Node)) {
    isOpen.value = false;
  }
}

onMounted(() => {
  document.addEventListener("click", handleClickOutside);
});

onUnmounted(() => {
  document.removeEventListener("click", handleClickOutside);
});
</script>

<template>
  <div
    ref="dropdownRef"
    class="layout-switcher-wrapper"
    :class="{ open: isOpen }"
  >
    <button class="layout-badge" @click="toggleDropdown" title="切换布局样式">
      <i class="fas fa-th-large badge-icon"></i>
      <span class="badge-text">{{ displayLabel }}</span>
      <i class="fas fa-caret-down switch-icon"></i>
    </button>

    <!-- 四宫格下拉菜单 -->
    <div
      ref="dropdownMenuRef"
      class="layout-dropdown"
      :class="{ 'align-right': alignRight }"
    >
      <div class="layout-grid">
        <button
          v-for="option in layoutOptions"
          :key="option.value"
          class="layout-option"
          :class="{ active: currentLayout === option.value }"
          @click="selectLayout(option.value)"
        >
          <!-- 布局预览图标 - Docker 专属设计 -->
          <div class="layout-preview" :class="`preview-${option.icon}`">
            <template v-if="option.icon === 'detail'">
              <div class="preview-detail">
                <div class="preview-header">
                  <div class="preview-icon-box"></div>
                  <div class="preview-info">
                    <div class="preview-line"></div>
                    <div class="preview-badge"></div>
                  </div>
                </div>
                <div class="preview-stats">
                  <div class="preview-stat-box"></div>
                  <div class="preview-stat-box"></div>
                </div>
              </div>
            </template>
            <template v-else-if="option.icon === 'compact'">
              <div class="preview-compact">
                <div class="preview-compact-item">
                  <div class="preview-dot"></div>
                  <div class="preview-line-short"></div>
                </div>
                <div class="preview-compact-item">
                  <div class="preview-dot"></div>
                  <div class="preview-line-short"></div>
                </div>
                <div class="preview-compact-item">
                  <div class="preview-dot"></div>
                  <div class="preview-line-short"></div>
                </div>
                <div class="preview-compact-item">
                  <div class="preview-dot"></div>
                  <div class="preview-line-short"></div>
                </div>
              </div>
            </template>
            <template v-else-if="option.icon === 'list'">
              <div class="preview-list">
                <div class="preview-list-row">
                  <div class="preview-list-dot"></div>
                  <div class="preview-list-line"></div>
                  <div class="preview-list-badge"></div>
                </div>
                <div class="preview-list-row">
                  <div class="preview-list-dot"></div>
                  <div class="preview-list-line"></div>
                  <div class="preview-list-badge"></div>
                </div>
              </div>
            </template>
            <template v-else-if="option.icon === 'minimal'">
              <div class="preview-minimal">
                <div class="preview-mini-card">
                  <div class="preview-mini-icon"></div>
                  <div class="preview-mini-dot"></div>
                </div>
                <div class="preview-mini-card">
                  <div class="preview-mini-icon"></div>
                  <div class="preview-mini-dot"></div>
                </div>
                <div class="preview-mini-card">
                  <div class="preview-mini-icon"></div>
                  <div class="preview-mini-dot"></div>
                </div>
                <div class="preview-mini-card">
                  <div class="preview-mini-icon"></div>
                  <div class="preview-mini-dot"></div>
                </div>
                <div class="preview-mini-card">
                  <div class="preview-mini-icon"></div>
                  <div class="preview-mini-dot"></div>
                </div>
                <div class="preview-mini-card">
                  <div class="preview-mini-icon"></div>
                  <div class="preview-mini-dot"></div>
                </div>
              </div>
            </template>
          </div>
          <span class="layout-label">{{ option.label }}</span>
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.layout-switcher-wrapper {
  position: relative;
  --local-color: 25 95% 53%; /* Orange */
}

/* 布局切换徽章按钮 - Docker 统一毛玻璃风格 */
.layout-badge {
  display: flex;
  align-items: center;
  gap: 6px;
  height: 40px;
  padding: 0 16px;
  border-radius: 12px;
  font-size: 14px;
  font-weight: 500;
  border: 1px solid hsl(var(--glass-border));
  cursor: pointer;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  background: hsl(var(--glass-bg));
  backdrop-filter: blur(var(--glass-blur));
  -webkit-backdrop-filter: blur(var(--glass-blur));
  color: hsl(var(--local-color));
  box-shadow: var(--shadow-sm), var(--glass-border-glow);
}

.layout-badge:hover {
  background: hsl(var(--glass-bg));
  border-color: hsl(var(--local-color) / 0.4);
  box-shadow: 
    var(--shadow-sm), 
    0 4px 12px -2px hsl(var(--local-color) / 0.15);
}

.badge-icon {
  font-size: 14px;
}

.switch-icon {
  font-size: 12px;
  opacity: 0.7;
  margin-left: 2px;
  transition: transform 0.25s ease;
}

.layout-switcher-wrapper.open .switch-icon {
  transform: rotate(180deg);
}

/* 下拉菜单 */
.layout-dropdown {
  position: absolute;
  top: calc(100% + 8px);
  left: 50%;
  background: hsl(var(--bg-elevated) / 0.85);
  backdrop-filter: blur(var(--glass-blur, 30px)) saturate(1.8);
  -webkit-backdrop-filter: blur(var(--glass-blur, 30px)) saturate(1.8);
  border: 1px solid hsl(var(--glass-border) / 0.8);
  border-radius: 16px;
  box-shadow: 
    var(--shadow-xl), 
    inset 0 1px 0 hsl(0 0% 100% / 0.1);
  padding: 10px;
  display: flex;
  gap: 8px;
  opacity: 0;
  visibility: hidden;
  transform: translateX(-50%) scale(0.95);
  transform-origin: top center;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 9999;
}

.layout-switcher-wrapper.open .layout-dropdown {
  opacity: 1;
  visibility: visible;
  transform: translateX(-50%) scale(1);
}

/* 右对齐模式 - 当下拉菜单会超出右边界时 */
.layout-dropdown.align-right {
  left: auto;
  right: 0;
  transform: scale(0.95);
  transform-origin: top right;
}

.layout-switcher-wrapper.open .layout-dropdown.align-right {
  transform: scale(1);
}

/* 四宫格布局 */
.layout-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 8px;
}

/* 布局选项 */
.layout-option {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  width: 72px;
  padding: 8px 6px;
  border: none;
  background: transparent;
  border-radius: 12px;
  color: hsl(var(--text-secondary));
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
}

.layout-option:hover {
  background: hsl(var(--primary) / 0.08);
  color: hsl(var(--text-primary));
}

.layout-option.active {
  background: hsl(var(--local-color) / 0.1);
  color: hsl(var(--local-color));
  font-weight: 500;
}

.layout-option.active .layout-preview {
  border-color: hsl(var(--local-color) / 0.3);
  background: hsl(var(--local-color) / 0.05);
}

/* 布局预览框 */
.layout-preview {
  width: 48px;
  height: 36px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.15);
  background: rgba(0, 0, 0, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 3px;
  transition: all 0.15s ease;
  overflow: hidden;
}

/* 详情布局预览 */
.preview-detail {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.preview-header {
  display: flex;
  align-items: center;
  gap: 3px;
}

.preview-icon-box {
  width: 8px;
  height: 8px;
  border-radius: 2px;
  background: rgba(249, 115, 22, 0.6);
  flex-shrink: 0;
}

.preview-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.preview-line {
  height: 2px;
  width: 100%;
  border-radius: 1px;
  background: rgba(255, 255, 255, 0.3);
}

.preview-badge {
  height: 2px;
  width: 50%;
  border-radius: 1px;
  background: rgba(0, 255, 128, 0.6);
}

.preview-stats {
  display: flex;
  gap: 2px;
  flex: 1;
}

.preview-stat-box {
  flex: 1;
  border-radius: 2px;
  background: rgba(249, 115, 22, 0.2);
  border: 1px solid rgba(249, 115, 22, 0.25);
}

/* 紧凑布局预览 */
.preview-compact {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 2px;
  width: 100%;
  height: 100%;
}

.preview-compact-item {
  display: flex;
  align-items: center;
  gap: 2px;
  padding: 2px;
  border-radius: 2px;
  background: rgba(249, 115, 22, 0.15);
}

.preview-dot {
  width: 4px;
  height: 4px;
  border-radius: 1px;
  background: rgba(249, 115, 22, 0.6);
  flex-shrink: 0;
}

.preview-line-short {
  flex: 1;
  height: 2px;
  border-radius: 1px;
  background: rgba(255, 255, 255, 0.25);
}

/* 列表布局预览 */
.preview-list {
  display: flex;
  flex-direction: column;
  gap: 3px;
  width: 100%;
}

.preview-list-row {
  display: flex;
  align-items: center;
  gap: 3px;
  padding: 2px 3px;
  border-radius: 2px;
  background: rgba(249, 115, 22, 0.15);
}

.preview-list-dot {
  width: 5px;
  height: 5px;
  border-radius: 1px;
  background: rgba(249, 115, 22, 0.6);
  flex-shrink: 0;
}

.preview-list-line {
  flex: 1;
  height: 2px;
  border-radius: 1px;
  background: rgba(255, 255, 255, 0.25);
}

.preview-list-badge {
  width: 8px;
  height: 3px;
  border-radius: 1px;
  background: rgba(0, 255, 128, 0.6);
  flex-shrink: 0;
}

/* 极简布局预览 */
.preview-minimal {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 2px;
  width: 100%;
  height: 100%;
}

.preview-mini-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1px;
  border-radius: 2px;
  background: rgba(249, 115, 22, 0.15);
  position: relative;
}

.preview-mini-icon {
  width: 5px;
  height: 5px;
  border-radius: 1px;
  background: rgba(249, 115, 22, 0.5);
}

.preview-mini-dot {
  width: 3px;
  height: 3px;
  border-radius: 50%;
  background: rgba(0, 255, 128, 0.7);
}

/* 布局标签 */
.layout-label {
  font-size: 11px;
  font-weight: 500;
  white-space: nowrap;
}

/* 浅色主题适配 */
[data-theme="light"] .layout-preview {
  border-color: rgba(0, 0, 0, 0.1);
  background: rgba(0, 0, 0, 0.03);
}

[data-theme="light"] .preview-line,
[data-theme="light"] .preview-line-short,
[data-theme="light"] .preview-list-line {
  background: rgba(0, 0, 0, 0.2);
}

/* 素描浅色主题适配 */
[data-theme="sketch-light"] .layout-preview {
  border-color: rgba(0, 0, 0, 0.12);
  background: rgba(0, 0, 0, 0.04);
}

[data-theme="sketch-light"] .preview-icon-box,
[data-theme="sketch-light"] .preview-dot,
[data-theme="sketch-light"] .preview-list-dot,
[data-theme="sketch-light"] .preview-mini-icon {
  background: rgba(60, 50, 40, 0.35);
}

[data-theme="sketch-light"] .preview-line,
[data-theme="sketch-light"] .preview-line-short,
[data-theme="sketch-light"] .preview-list-line {
  background: rgba(60, 50, 40, 0.2);
}

[data-theme="sketch-light"] .preview-badge,
[data-theme="sketch-light"] .preview-list-badge,
[data-theme="sketch-light"] .preview-mini-dot {
  background: rgba(60, 50, 40, 0.4);
}

[data-theme="sketch-light"] .preview-stat-box {
  background: rgba(60, 50, 40, 0.08);
  border-color: rgba(60, 50, 40, 0.15);
}

[data-theme="sketch-light"] .preview-compact-item,
[data-theme="sketch-light"] .preview-list-row,
[data-theme="sketch-light"] .preview-mini-card {
  background: rgba(60, 50, 40, 0.08);
}

/* 素描深色主题适配 */
[data-theme="sketch-dark"] .layout-option.active::after {
  background: rgba(255, 255, 255, 0.5);
  box-shadow: 0 0 6px rgba(255, 255, 255, 0.3);
}

[data-theme="sketch-dark"] .layout-preview {
  border-color: rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.03);
}

[data-theme="sketch-dark"] .preview-icon-box,
[data-theme="sketch-dark"] .preview-dot,
[data-theme="sketch-dark"] .preview-list-dot,
[data-theme="sketch-dark"] .preview-mini-icon {
  background: rgba(255, 255, 255, 0.25);
}

[data-theme="sketch-dark"] .preview-line,
[data-theme="sketch-dark"] .preview-line-short,
[data-theme="sketch-dark"] .preview-list-line {
  background: rgba(255, 255, 255, 0.18);
}

[data-theme="sketch-dark"] .preview-badge,
[data-theme="sketch-dark"] .preview-list-badge,
[data-theme="sketch-dark"] .preview-mini-dot {
  background: rgba(255, 255, 255, 0.3);
}

[data-theme="sketch-dark"] .preview-stat-box {
  background: rgba(255, 255, 255, 0.05);
  border-color: rgba(255, 255, 255, 0.1);
}

[data-theme="sketch-dark"] .preview-compact-item,
[data-theme="sketch-dark"] .preview-list-row,
[data-theme="sketch-dark"] .preview-mini-card {
  background: rgba(255, 255, 255, 0.06);
}



/* 响应式 */
@media (max-width: 640px) {
  .layout-badge {
    padding: 5px 10px;
    font-size: 12px;
  }

  .badge-text {
    max-width: 48px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .layout-option {
    width: 64px;
    padding: 8px 6px;
  }

  .layout-preview {
    width: 42px;
    height: 32px;
  }
}
</style>
