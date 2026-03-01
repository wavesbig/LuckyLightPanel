<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from "vue";
import type { Group } from "@/types";

const props = defineProps<{
  groups: Group[];
  current: string | string[]; // 支持单选或多选
  color?: "cyan" | "docker" | "green";
}>();

const emit = defineEmits<{
  change: [key: string | string[]]; // 单选/设置分组
  toggle: [key: string]; // 切换单个分组（多选模式）
}>();

const isOpen = ref(false);
const dropdownRef = ref<HTMLElement | null>(null);

// 当前选中的分组（规范化为数组）
const selectedGroups = computed<string[]>(() => {
  const current = props.current;
  if (current === "all") return [];
  if (Array.isArray(current)) return current;
  return [current];
});

// 是否选中“全部”
const isAllSelected = computed(() => {
  return props.current === "all" || selectedGroups.value.length === 0;
});

// 检查分组是否被选中
function isGroupSelected(key: string): boolean {
  if (key === "all") return isAllSelected.value;
  return selectedGroups.value.includes(key);
}

// 获取显示文本
const displayText = computed(() => {
  if (isAllSelected.value) return "全部";
  const count = selectedGroups.value.length;
  if (count === 1) {
    const group = props.groups.find((g) => g.key === selectedGroups.value[0]);
    return group?.name || "全部";
  }
  return `已选 ${count} 个分组`;
});

// 获取显示图标
const displayIcon = computed(() => {
  if (isAllSelected.value || selectedGroups.value.length !== 1) {
    return "fas fa-layer-group";
  }
  const group = props.groups.find((g) => g.key === selectedGroups.value[0]);
  return getGroupIconClass(group?.icon);
});

// 获取分组图标类名
function getGroupIconClass(icon?: string): string {
  if (!icon) return "fas fa-folder";
  if (
    icon.startsWith("fa-") ||
    icon.startsWith("fas ") ||
    icon.startsWith("far ") ||
    icon.startsWith("fab ")
  ) {
    return icon.startsWith("fa-") ? `fas ${icon}` : icon;
  }
  return `fas fa-${icon}`;
}

// 切换下拉菜单
function toggleDropdown() {
  isOpen.value = !isOpen.value;
}

// 点击分组名称（单选模式）- 直接切换到该分组
function onGroupClick(key: string) {
  // 单选行为：直接设置为该分组
  if (key === "all") {
    emit("change", "all");
  } else {
    emit("change", [key]);
  }
  isOpen.value = false;
}

// 点击复选框（多选模式）- 切换选中状态
function onCheckboxClick(key: string, event: MouseEvent) {
  event.stopPropagation();
  emit("toggle", key);
  // 多选模式不关闭下拉菜单，方便继续选择
  // 但如果点击"全部"则关闭
  if (key === "all") {
    isOpen.value = false;
  }
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
    v-if="groups.length > 1"
    ref="dropdownRef"
    class="group-dropdown-wrapper"
    :class="[`color-${color || 'cyan'}`, { open: isOpen }]"
  >
    <button class="group-badge" @click="toggleDropdown">
      <i :class="displayIcon" class="badge-icon"></i>
      <span class="badge-text">{{ displayText }}</span>
      <i class="fas fa-caret-down switch-icon"></i>
    </button>

    <div class="group-dropdown">
      <!-- 全部选项 -->
      <button
        class="group-option"
        :class="{ active: isAllSelected }"
        @click="onGroupClick('all')"
      >
        <i class="fas fa-layer-group option-icon"></i>
        <span>全部</span>
      </button>

      <div class="dropdown-divider"></div>

      <!-- 其他分组选项 -->
      <div
        v-for="group in groups.filter((g) => g.key !== 'all')"
        :key="group.key"
        class="group-option"
        :class="{ active: isGroupSelected(group.key) }"
      >
        <!-- 左侧：点击切换单选 -->
        <button class="option-main" @click="onGroupClick(group.key)">
          <i :class="getGroupIconClass(group.icon)" class="option-icon"></i>
          <span class="option-name">{{ group.name }}</span>
          <span v-if="group.count !== undefined" class="option-count">{{
            group.count
          }}</span>
        </button>
        <!-- 右侧：复选框，点击切换多选 -->
        <button
          class="option-checkbox"
          :class="{ checked: isGroupSelected(group.key) }"
          @click="onCheckboxClick(group.key, $event)"
          :title="isGroupSelected(group.key) ? '取消选中' : '添加到选中'"
        >
          <i v-if="isGroupSelected(group.key)" class="fas fa-check-square"></i>
          <i v-else class="far fa-square"></i>
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.group-dropdown-wrapper {
  position: relative;
}

/* 分组徽章按钮 - 统一毛玻璃风格 */
.group-badge {
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
  color: hsl(var(--text-secondary));
  box-shadow: var(--shadow-sm), var(--glass-border-glow);
}

.group-badge:hover {
  background: hsl(var(--glass-bg));
  color: hsl(var(--text-primary));
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

.group-badge:hover .switch-icon {
  opacity: 0.8;
}

.group-dropdown-wrapper.open .switch-icon {
  transform: rotate(180deg);
}

/* 颜色重载 - 边框和文字色彩 */
.color-cyan { --local-color: 188 86% 53%; }  /* Cyan 500 */
.color-docker { --local-color: 206 84% 54%; } /* Blue 500 */
.color-green { --local-color: 160 84% 39%; }  /* Emerald 600 */

.color-cyan .group-badge, .color-docker .group-badge, .color-green .group-badge {
  color: hsl(var(--local-color, var(--primary)));
}

.color-cyan .group-badge:hover, .color-docker .group-badge:hover, .color-green .group-badge:hover {
  border-color: hsl(var(--local-color, var(--primary)) / 0.4);
  box-shadow: 
    var(--shadow-sm), 
    0 4px 12px -2px hsl(var(--local-color, var(--primary)) / 0.15);
}

.group-option.active {
  background: hsl(var(--local-color, var(--primary)) / 0.1);
  color: hsl(var(--local-color, var(--primary)));
  font-weight: 500;
}

/* 下拉菜单 */
.group-dropdown {
  position: absolute;
  top: calc(100% + 8px);
  left: 50%;
  min-width: 180px;
  max-width: 240px;
  max-height: 320px;
  overflow-y: auto;
  border-radius: 16px;
  background: hsl(var(--bg-elevated) / 0.85);
  backdrop-filter: blur(var(--glass-blur, 30px)) saturate(1.8);
  -webkit-backdrop-filter: blur(var(--glass-blur, 30px)) saturate(1.8);
  border: 1px solid hsl(var(--glass-border) / 0.8);
  box-shadow: 
    var(--shadow-xl), 
    inset 0 1px 0 hsl(0 0% 100% / 0.1);
  padding: 8px;
  opacity: 0;
  visibility: hidden;
  transform: translateX(-50%) scale(0.95);
  transform-origin: top center;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 9999;
}

.group-dropdown-wrapper.open .group-dropdown {
  opacity: 1;
  visibility: visible;
  transform: translateX(-50%) scale(1);
}

/* 下拉选项容器 */
.group-option {
  display: flex;
  align-items: center;
  width: 100%;
  border-radius: 8px;
  color: hsl(var(--text-secondary));
  font-size: 14px;
  transition: all 0.15s ease;
  overflow: hidden;
}

/* 全部选项（button形式） */
button.group-option {
  gap: 8px;
  padding: 8px 12px;
  border: none;
  background: transparent;
  cursor: pointer;
}

button.group-option:hover {
  background: hsl(var(--primary) / 0.08);
  color: hsl(var(--text-primary));
}

/* 分组选项主体（左侧可点击区域） */
.option-main {
  display: flex;
  align-items: center;
  gap: 8px;
  flex: 1;
  padding: 8px 8px 8px 12px;
  border: none;
  background: transparent;
  color: inherit;
  font-size: inherit;
  cursor: pointer;
  overflow: hidden;
}

.option-main:hover {
  background: hsl(var(--primary) / 0.08);
  color: hsl(var(--text-primary));
}

.option-icon {
  width: 16px;
  text-align: center;
  font-size: 12px;
  flex-shrink: 0;
}

.option-main span {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  text-align: left;
}

.option-name {
  flex: 1;
}

.option-count {
  flex-shrink: 0;
  font-size: 11px;
  color: hsl(var(--text-muted));
  background: hsl(var(--glass-border));
  padding: 1px 6px;
  border-radius: 8px;
  margin-left: 4px;
}

/* 右侧复选框 */
.option-checkbox {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 100%;
  min-height: 36px;
  padding: 0;
  border: none;
  background: transparent;
  color: hsl(var(--text-muted));
  font-size: 14px;
  cursor: pointer;
  transition: all 0.15s ease;
  flex-shrink: 0;
}

.option-checkbox:hover {
  background: hsl(var(--primary) / 0.08);
  color: hsl(var(--text-primary));
}

.option-checkbox.checked {
  color: hsl(var(--local-color, var(--primary)));
}

/* 分割线 */
.dropdown-divider {
  height: 1px;
  background: hsl(var(--glass-border) / 0.8);
  margin: 4px 8px;
}



/* 响应式 */
@media (max-width: 640px) {
  .group-badge {
    padding: 5px 10px;
    font-size: 12px;
  }

  .badge-text {
    max-width: 80px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
}
</style>
