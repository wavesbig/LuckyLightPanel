<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed, nextTick } from "vue";
import { useConfigStore } from "@/stores/config";
import type { LayoutMode } from "@/types";

const configStore = useConfigStore();

const isOpen = ref(false);
const dropdownRef = ref<HTMLElement | null>(null);
const dropdownMenuRef = ref<HTMLElement | null>(null);
const alignRight = ref(false);

// 布局选项配置 - 四种布局
const layoutOptions: {
  value: LayoutMode;
  label: string;
  icon: string;
  desc: string;
}[] = [
  { value: "normal", label: "卡片", icon: "card", desc: "标准布局" },
  { value: "compact", label: "紧凑", icon: "compact", desc: "显示更多" },
  { value: "list", label: "列表", icon: "list", desc: "横向展示" },
  { value: "minimal", label: "极简", icon: "minimal", desc: "图标导航" },
];

// 当前选中的布局
const currentLayout = computed(() => configStore.layout);

// 获取当前布局的显示名称
const displayLabel = computed(() => {
  const option = layoutOptions.find((o) => o.value === currentLayout.value);
  return option?.label || "卡片";
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
  configStore.setLayout(layout);
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
          <!-- 布局预览图标 -->
          <div class="layout-preview" :class="`preview-${option.icon}`">
            <template v-if="option.icon === 'card'">
              <div class="preview-card">
                <div class="preview-card-icon">
                  <div class="card-icon-inner"></div>
                </div>
                <div class="preview-card-content">
                  <div class="card-title-bar"></div>
                  <div class="card-desc-bar"></div>
                  <div class="card-meta">
                    <div class="card-status-dot"></div>
                    <div class="card-badge"></div>
                  </div>
                </div>
              </div>
            </template>
            <template v-else-if="option.icon === 'compact'">
              <div class="preview-compact">
                <div class="compact-item">
                  <div class="compact-icon"></div>
                  <div class="compact-lines">
                    <div class="compact-line"></div>
                    <div class="compact-line short"></div>
                  </div>
                </div>
                <div class="compact-item">
                  <div class="compact-icon"></div>
                  <div class="compact-lines">
                    <div class="compact-line"></div>
                    <div class="compact-line short"></div>
                  </div>
                </div>
                <div class="compact-item">
                  <div class="compact-icon"></div>
                  <div class="compact-lines">
                    <div class="compact-line"></div>
                    <div class="compact-line short"></div>
                  </div>
                </div>
                <div class="compact-item">
                  <div class="compact-icon"></div>
                  <div class="compact-lines">
                    <div class="compact-line"></div>
                    <div class="compact-line short"></div>
                  </div>
                </div>
              </div>
            </template>
            <template v-else-if="option.icon === 'list'">
              <div class="preview-list">
                <div class="list-row">
                  <div class="list-icon"></div>
                  <div class="list-text"></div>
                  <div class="list-badge"></div>
                </div>
                <div class="list-row">
                  <div class="list-icon"></div>
                  <div class="list-text"></div>
                  <div class="list-badge"></div>
                </div>
                <div class="list-row">
                  <div class="list-icon"></div>
                  <div class="list-text"></div>
                  <div class="list-badge"></div>
                </div>
              </div>
            </template>
            <template v-else-if="option.icon === 'minimal'">
              <div class="preview-minimal">
                <div class="minimal-cell">
                  <div class="minimal-icon"></div>
                  <div class="minimal-dot"></div>
                </div>
                <div class="minimal-cell">
                  <div class="minimal-icon"></div>
                  <div class="minimal-dot"></div>
                </div>
                <div class="minimal-cell">
                  <div class="minimal-icon"></div>
                  <div class="minimal-dot"></div>
                </div>
                <div class="minimal-cell">
                  <div class="minimal-icon"></div>
                  <div class="minimal-dot"></div>
                </div>
                <div class="minimal-cell">
                  <div class="minimal-icon"></div>
                  <div class="minimal-dot"></div>
                </div>
                <div class="minimal-cell">
                  <div class="minimal-icon"></div>
                  <div class="minimal-dot"></div>
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
  --local-color: 188 86% 53%; /* Cyan */
}

/* 布局切换徽章按钮 - 统一毛玻璃风格 */
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
  font-size: 16px;
}

.switch-icon {
  font-size: 12px;
  opacity: 0.5;
  margin-left: 2px;
  transition: transform 0.25s ease, opacity 0.25s ease;
}

.layout-badge:hover .switch-icon {
  opacity: 0.8;
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
  width: 76px;
  padding: 8px 6px;
  border: 1px solid transparent;
  background: transparent;
  border-radius: 12px;
  color: hsl(var(--text-secondary));
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
}

.layout-option::before {
  content: "";
  position: absolute;
  inset: 0;
  border-radius: inherit;
  background: linear-gradient(
    135deg,
    transparent 0%,
    hsl(var(--local-color) / 0.1) 100%
  );
  opacity: 0;
  transition: opacity 0.2s ease;
  pointer-events: none;
}

.layout-option:hover {
  background: hsl(var(--primary) / 0.08);
  border-color: hsl(var(--glass-border));
  color: hsl(var(--text-primary));
}

.layout-option:hover::before {
  opacity: 1;
}

.layout-option.active {
  background: hsl(var(--local-color) / 0.1);
  border-color: transparent;
  color: hsl(var(--local-color));
  font-weight: 500;
}

.layout-option.active .layout-preview {
  border-color: hsl(var(--local-color) / 0.3);
  background: hsl(var(--local-color) / 0.05);
}

.layout-option.active .layout-preview::before {
  background: linear-gradient(
    135deg,
    rgba(6, 182, 212, 0.1) 0%,
    transparent 50%
  );
}

/* 布局预览框 */
.layout-preview {
  width: 52px;
  height: 40px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.12);
  background: linear-gradient(
    135deg,
    rgba(0, 0, 0, 0.25) 0%,
    rgba(0, 0, 0, 0.15) 100%
  );
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 5px;
  transition: all 0.2s ease;
  overflow: hidden;
  position: relative;
}

.layout-preview::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.05) 0%,
    transparent 50%
  );
  pointer-events: none;
}

.layout-option:hover .layout-preview {
  transform: scale(1.05);
  border-color: rgba(6, 182, 212, 0.3);
}

/* ============ 卡片布局预览 ============ */
.preview-card {
  display: flex;
  align-items: flex-start;
  gap: 5px;
  width: 100%;
  height: 100%;
}

.preview-card-icon {
  width: 14px;
  height: 14px;
  border-radius: 4px;
  background: linear-gradient(
    135deg,
    rgba(6, 182, 212, 0.7) 0%,
    rgba(139, 92, 246, 0.5) 100%
  );
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 4px rgba(6, 182, 212, 0.3);
}

.card-icon-inner {
  width: 6px;
  height: 6px;
  border-radius: 1px;
  background: rgba(255, 255, 255, 0.8);
}

.preview-card-content {
  display: flex;
  flex-direction: column;
  gap: 3px;
  flex: 1;
  min-width: 0;
}

.card-title-bar {
  height: 4px;
  width: 100%;
  border-radius: 2px;
  background: rgba(255, 255, 255, 0.4);
}

.card-desc-bar {
  height: 3px;
  width: 70%;
  border-radius: 1.5px;
  background: rgba(255, 255, 255, 0.2);
}

.card-meta {
  display: flex;
  align-items: center;
  gap: 3px;
  margin-top: 1px;
}

.card-status-dot {
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: #22c55e;
  box-shadow: 0 0 4px rgba(34, 197, 94, 0.6);
}

.card-badge {
  height: 3px;
  width: 12px;
  border-radius: 1.5px;
  background: rgba(6, 182, 212, 0.4);
}

/* ============ 紧凑布局预览 ============ */
.preview-compact {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 3px;
  width: 100%;
  height: 100%;
}

.compact-item {
  display: flex;
  align-items: center;
  gap: 3px;
  padding: 2px;
  border-radius: 3px;
  background: rgba(255, 255, 255, 0.05);
}

.compact-icon {
  width: 8px;
  height: 8px;
  border-radius: 2px;
  background: linear-gradient(
    135deg,
    rgba(6, 182, 212, 0.6) 0%,
    rgba(139, 92, 246, 0.4) 100%
  );
  flex-shrink: 0;
}

.compact-lines {
  display: flex;
  flex-direction: column;
  gap: 2px;
  flex: 1;
  min-width: 0;
}

.compact-line {
  height: 2px;
  border-radius: 1px;
  background: rgba(255, 255, 255, 0.25);
}

.compact-line.short {
  width: 60%;
}

/* ============ 列表布局预览 ============ */
.preview-list {
  display: flex;
  flex-direction: column;
  gap: 3px;
  width: 100%;
  height: 100%;
  justify-content: center;
}

.list-row {
  display: flex;
  align-items: center;
  gap: 3px;
  padding: 2px 3px;
  border-radius: 3px;
  background: rgba(255, 255, 255, 0.04);
}

.list-icon {
  width: 6px;
  height: 6px;
  border-radius: 2px;
  background: linear-gradient(
    135deg,
    rgba(6, 182, 212, 0.6) 0%,
    rgba(139, 92, 246, 0.4) 100%
  );
  flex-shrink: 0;
}

.list-text {
  flex: 1;
  height: 3px;
  border-radius: 1.5px;
  background: rgba(255, 255, 255, 0.25);
}

.list-badge {
  width: 8px;
  height: 3px;
  border-radius: 1.5px;
  background: rgba(34, 197, 94, 0.5);
  flex-shrink: 0;
}

/* ============ 极简布局预览 ============ */
.preview-minimal {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 3px;
  width: 100%;
  height: 100%;
}

.minimal-cell {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 2px;
  position: relative;
}

.minimal-icon {
  width: 9px;
  height: 9px;
  border-radius: 3px;
  background: linear-gradient(
    135deg,
    rgba(6, 182, 212, 0.5) 0%,
    rgba(139, 92, 246, 0.3) 100%
  );
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

.minimal-dot {
  width: 3px;
  height: 3px;
  border-radius: 50%;
  background: #22c55e;
  box-shadow: 0 0 3px rgba(34, 197, 94, 0.5);
}

/* 布局标签 */
.layout-label {
  font-size: 11px;
  font-weight: 500;
  white-space: nowrap;
}

/* ============ 浅色主题适配 ============ */
[data-theme="light"] .layout-preview {
  border-color: rgba(0, 0, 0, 0.1);
  background: linear-gradient(
    135deg,
    rgba(0, 0, 0, 0.04) 0%,
    rgba(0, 0, 0, 0.02) 100%
  );
}

[data-theme="light"] .layout-preview::before {
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.5) 0%,
    transparent 50%
  );
}

[data-theme="light"] .card-title-bar {
  background: rgba(0, 0, 0, 0.25);
}

[data-theme="light"] .card-desc-bar {
  background: rgba(0, 0, 0, 0.12);
}

[data-theme="light"] .card-badge {
  background: rgba(6, 182, 212, 0.5);
}

[data-theme="light"] .compact-item {
  background: rgba(0, 0, 0, 0.03);
}

[data-theme="light"] .compact-line {
  background: rgba(0, 0, 0, 0.15);
}

[data-theme="light"] .list-row {
  background: rgba(0, 0, 0, 0.03);
}

[data-theme="light"] .list-text {
  background: rgba(0, 0, 0, 0.15);
}

[data-theme="light"] .minimal-icon {
  background: linear-gradient(
    135deg,
    rgba(6, 182, 212, 0.6) 0%,
    rgba(139, 92, 246, 0.4) 100%
  );
}

/* ============ 素描浅色主题适配 ============ */
[data-theme="sketch-light"] .layout-option {
  color: rgba(60, 50, 40, 0.85);
}

[data-theme="sketch-light"] .layout-option::before {
  background: linear-gradient(
    135deg,
    transparent 0%,
    rgba(60, 50, 40, 0.05) 100%
  );
}

[data-theme="sketch-light"] .layout-option:hover {
  background: rgba(0, 0, 0, 0.04);
  border-color: rgba(0, 0, 0, 0.06);
}

[data-theme="sketch-light"] .layout-option.active {
  background: linear-gradient(
    135deg,
    rgba(60, 50, 40, 0.08) 0%,
    rgba(80, 70, 60, 0.04) 100%
  );
  border-color: rgba(60, 50, 40, 0.25);
  color: rgba(60, 50, 40, 1);
  box-shadow:
    0 0 15px -5px rgba(60, 50, 40, 0.2),
    inset 0 1px 0 rgba(255, 255, 255, 0.4);
}

[data-theme="sketch-light"] .layout-option.active::after {
  background: rgba(60, 50, 40, 0.6);
  box-shadow: 0 0 6px rgba(60, 50, 40, 0.4);
}

[data-theme="sketch-light"] .layout-option.active .layout-preview {
  border-color: rgba(60, 50, 40, 0.3);
  background: linear-gradient(
    135deg,
    rgba(60, 50, 40, 0.06) 0%,
    rgba(80, 70, 60, 0.03) 100%
  );
  box-shadow: 0 0 10px -3px rgba(60, 50, 40, 0.2);
}

[data-theme="sketch-light"] .layout-preview {
  border-color: rgba(0, 0, 0, 0.12);
  background: linear-gradient(
    135deg,
    rgba(0, 0, 0, 0.05) 0%,
    rgba(0, 0, 0, 0.02) 100%
  );
}

[data-theme="sketch-light"] .layout-preview::before {
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.4) 0%,
    transparent 50%
  );
}

[data-theme="sketch-light"] .preview-card-icon {
  background: linear-gradient(
    135deg,
    rgba(60, 50, 40, 0.5) 0%,
    rgba(80, 70, 60, 0.3) 100%
  );
  box-shadow: 0 2px 4px rgba(60, 50, 40, 0.2);
}

[data-theme="sketch-light"] .card-title-bar {
  background: rgba(60, 50, 40, 0.3);
}

[data-theme="sketch-light"] .card-desc-bar {
  background: rgba(60, 50, 40, 0.15);
}

[data-theme="sketch-light"] .card-status-dot {
  background: rgba(60, 50, 40, 0.4);
  box-shadow: none;
}

[data-theme="sketch-light"] .card-badge {
  background: rgba(60, 50, 40, 0.25);
}

[data-theme="sketch-light"] .compact-item {
  background: rgba(0, 0, 0, 0.03);
}

[data-theme="sketch-light"] .compact-icon {
  background: linear-gradient(
    135deg,
    rgba(60, 50, 40, 0.4) 0%,
    rgba(80, 70, 60, 0.2) 100%
  );
}

[data-theme="sketch-light"] .compact-line {
  background: rgba(60, 50, 40, 0.2);
}

[data-theme="sketch-light"] .list-row {
  background: rgba(0, 0, 0, 0.03);
}

[data-theme="sketch-light"] .list-icon {
  background: linear-gradient(
    135deg,
    rgba(60, 50, 40, 0.4) 0%,
    rgba(80, 70, 60, 0.2) 100%
  );
}

[data-theme="sketch-light"] .list-text {
  background: rgba(60, 50, 40, 0.2);
}

[data-theme="sketch-light"] .list-badge {
  background: rgba(60, 50, 40, 0.25);
}

[data-theme="sketch-light"] .minimal-icon {
  background: linear-gradient(
    135deg,
    rgba(60, 50, 40, 0.35) 0%,
    rgba(80, 70, 60, 0.2) 100%
  );
  box-shadow: 0 1px 3px rgba(60, 50, 40, 0.15);
}

[data-theme="sketch-light"] .minimal-dot {
  background: rgba(60, 50, 40, 0.35);
  box-shadow: none;
}

/* ============ 素描深色主题适配 ============ */
[data-theme="sketch-dark"] .layout-dropdown {
  background: rgba(35, 30, 25, 0.95);
  border-color: rgba(255, 255, 255, 0.08);
  box-shadow:
    0 8px 32px rgba(0, 0, 0, 0.35),
    inset 0 1px 0 rgba(255, 255, 255, 0.03);
}

[data-theme="sketch-dark"] .layout-option {
  color: rgba(255, 255, 255, 0.8);
}

[data-theme="sketch-dark"] .layout-option::before {
  background: linear-gradient(
    135deg,
    transparent 0%,
    rgba(255, 255, 255, 0.03) 100%
  );
}

[data-theme="sketch-dark"] .layout-option:hover {
  background: rgba(255, 255, 255, 0.05);
  border-color: rgba(255, 255, 255, 0.06);
}

[data-theme="sketch-dark"] .layout-option.active {
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.08) 0%,
    rgba(255, 255, 255, 0.04) 100%
  );
  border-color: rgba(255, 255, 255, 0.2);
  color: rgba(255, 255, 255, 0.95);
  box-shadow:
    0 0 15px -5px rgba(255, 255, 255, 0.15),
    inset 0 1px 0 rgba(255, 255, 255, 0.05);
}

[data-theme="sketch-dark"] .layout-option.active::after {
  background: rgba(255, 255, 255, 0.5);
  box-shadow: 0 0 6px rgba(255, 255, 255, 0.3);
}

[data-theme="sketch-dark"] .layout-option.active .layout-preview {
  border-color: rgba(255, 255, 255, 0.2);
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.06) 0%,
    rgba(255, 255, 255, 0.03) 100%
  );
  box-shadow: 0 0 10px -3px rgba(255, 255, 255, 0.15);
}

[data-theme="sketch-dark"] .layout-preview {
  border-color: rgba(255, 255, 255, 0.1);
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.04) 0%,
    rgba(255, 255, 255, 0.02) 100%
  );
}

[data-theme="sketch-dark"] .layout-preview::before {
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.03) 0%,
    transparent 50%
  );
}

[data-theme="sketch-dark"] .preview-card-icon {
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.2) 0%,
    rgba(255, 255, 255, 0.1) 100%
  );
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

[data-theme="sketch-dark"] .card-title-bar {
  background: rgba(255, 255, 255, 0.25);
}

[data-theme="sketch-dark"] .card-desc-bar {
  background: rgba(255, 255, 255, 0.12);
}

[data-theme="sketch-dark"] .card-status-dot {
  background: rgba(255, 255, 255, 0.3);
  box-shadow: none;
}

[data-theme="sketch-dark"] .card-badge {
  background: rgba(255, 255, 255, 0.15);
}

[data-theme="sketch-dark"] .compact-item {
  background: rgba(255, 255, 255, 0.03);
}

[data-theme="sketch-dark"] .compact-icon {
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.2) 0%,
    rgba(255, 255, 255, 0.1) 100%
  );
}

[data-theme="sketch-dark"] .compact-line {
  background: rgba(255, 255, 255, 0.15);
}

[data-theme="sketch-dark"] .list-row {
  background: rgba(255, 255, 255, 0.03);
}

[data-theme="sketch-dark"] .list-icon {
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.2) 0%,
    rgba(255, 255, 255, 0.1) 100%
  );
}

[data-theme="sketch-dark"] .list-text {
  background: rgba(255, 255, 255, 0.15);
}

[data-theme="sketch-dark"] .list-badge {
  background: rgba(255, 255, 255, 0.18);
}

[data-theme="sketch-dark"] .minimal-icon {
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.18) 0%,
    rgba(255, 255, 255, 0.08) 100%
  );
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.25);
}

[data-theme="sketch-dark"] .minimal-dot {
  background: rgba(255, 255, 255, 0.25);
  box-shadow: none;
}

/* 深色主题适配 */
[data-theme="dark"] .layout-dropdown {
  box-shadow:
    0 8px 32px rgba(0, 0, 0, 0.4),
    inset 0 1px 0 rgba(255, 255, 255, 0.03);
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
