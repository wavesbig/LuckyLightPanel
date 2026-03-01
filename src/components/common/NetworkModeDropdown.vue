<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from "vue";
import { useConfigStore } from "@/stores/config";
import { useNavStore } from "@/stores/nav";
import type { NetworkMode } from "@/types";

const configStore = useConfigStore();
const navStore = useNavStore();

const isOpen = ref(false);
const dropdownRef = ref<HTMLElement | null>(null);

// 网络模式选项
const networkModeOptions: {
  value: NetworkMode;
  label: string;
  icon: string;
}[] = [
  { value: "auto", label: "自动", icon: "fas fa-magic" },
  { value: "internal", label: "内网", icon: "fas fa-home" },
  { value: "external", label: "外网", icon: "fas fa-globe" },
  { value: "hybrid", label: "混合", icon: "fas fa-random" },
];

// 获取当前显示的文本（自动模式显示识别结果）
const displayLabel = computed(() => {
  const mode = configStore.networkMode;
  if (mode === "auto") {
    // 自动模式：查询失败时显示"自动(混合)"，否则显示识别结果
    if (navStore.networkTypeFetchFailed) {
      return "自动(混合)";
    }
    const isInternal = navStore.networkType === "internal";
    return isInternal ? "自动(内网)" : "自动(外网)";
  }
  const option = networkModeOptions.find((o) => o.value === mode);
  return option?.label || "混合";
});

// 获取当前显示的图标
const displayIcon = computed(() => {
  const mode = configStore.networkMode;
  if (mode === "auto") {
    return "fas fa-magic";
  }
  const option = networkModeOptions.find((o) => o.value === mode);
  return option?.icon || "fas fa-random";
});

// 切换下拉菜单
function toggleDropdown() {
  isOpen.value = !isOpen.value;
}

// 选择模式
async function selectMode(mode: NetworkMode) {
  configStore.setNetworkMode(mode);
  isOpen.value = false;

  // 切换到自动或混合模式时，重新获取网络类型
  if (mode === "auto" || mode === "hybrid") {
    await navStore.fetchNetworkType();
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
    ref="dropdownRef"
    class="network-dropdown-wrapper"
    :class="{ open: isOpen }"
  >
    <button class="network-badge" @click="toggleDropdown" title="切换网络模式">
      <i :class="displayIcon" class="badge-icon"></i>
      <span class="badge-text">{{ displayLabel }}</span>
      <i class="fas fa-caret-down switch-icon"></i>
    </button>

    <div class="network-dropdown">
      <button
        v-for="option in networkModeOptions"
        :key="option.value"
        class="network-option"
        :class="{ active: configStore.networkMode === option.value }"
        @click="selectMode(option.value)"
      >
        <i :class="option.icon" class="option-icon"></i>
        <span>{{ option.label }}</span>
      </button>
    </div>
  </div>
</template>

<style scoped>
.network-dropdown-wrapper {
  position: relative;
  --local-color: 285 89% 65%; /* purple 500 */
}

/* 网络模式徽章按钮 - 统一毛玻璃风格 */
.network-badge {
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

.network-badge:hover {
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

.network-badge:hover .switch-icon {
  opacity: 0.8;
}

.network-dropdown-wrapper.open .switch-icon {
  transform: rotate(180deg);
}

/* 下拉菜单 */
.network-dropdown {
  position: absolute;
  top: calc(100% + 8px);
  left: 50%;
  min-width: 140px;
  max-width: 180px;
  background: hsl(var(--bg-elevated) / 0.85);
  backdrop-filter: blur(var(--glass-blur, 30px)) saturate(1.8);
  -webkit-backdrop-filter: blur(var(--glass-blur, 30px)) saturate(1.8);
  border: 1px solid hsl(var(--glass-border) / 0.8);
  border-radius: 16px;
  box-shadow: 
    var(--shadow-xl), 
    inset 0 1px 0 hsl(0 0% 100% / 0.1);
  padding: 8px;
  display: flex;
  flex-direction: column;
  gap: 4px;
  opacity: 0;
  visibility: hidden;
  transform: translateX(-50%) scale(0.95);
  transform-origin: top center;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 9999;
}

.network-dropdown-wrapper.open .network-dropdown {
  opacity: 1;
  visibility: visible;
  transform: translateX(-50%) scale(1);
}

/* 下拉选项 */
.network-option {
  display: flex;
  align-items: center;
  gap: 8px;
  width: 100%;
  padding: 8px 12px;
  border: none;
  background: transparent;
  border-radius: 8px;
  color: hsl(var(--text-secondary));
  font-size: 14px;
  cursor: pointer;
  transition: all 0.15s ease;
  white-space: nowrap;
}

.network-option:hover {
  background: hsl(var(--primary) / 0.08);
  color: hsl(var(--text-primary));
}

.network-option.active {
  background: hsl(var(--local-color) / 0.1);
  color: hsl(var(--local-color));
  font-weight: 500;
}

.option-icon {
  width: 16px;
  text-align: center;
  font-size: 14px;
  flex-shrink: 0;
}



/* 响应式 */
@media (max-width: 640px) {
  .network-badge {
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
