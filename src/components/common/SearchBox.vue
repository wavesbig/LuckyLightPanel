<script setup lang="ts">
import { computed, ref, watch } from "vue";

const props = withDefaults(
  defineProps<{
    modelValue: string;
    placeholder?: string;
    color?: "cyan" | "docker" | "green";
  }>(),
  {
    placeholder: "搜索...",
    color: "cyan",
  },
);

const emit = defineEmits<{
  (e: "update:modelValue", value: string): void;
}>();

// 输入框引用
const inputRef = ref<HTMLInputElement | null>(null);

// 本地值（用于输入防抖）
const localValue = ref(props.modelValue);

// 监听外部值变化
watch(
  () => props.modelValue,
  (newVal) => {
    localValue.value = newVal;
  },
);

// 是否显示清除按钮
const showClear = computed(() => localValue.value.length > 0);

// 颜色主题
const colorClass = computed(() => {
  switch (props.color) {
    case "docker":
      return "search-docker";
    case "green":
      return "search-green";
    default:
      return "search-cyan";
  }
});

// 输入处理
function handleInput(e: Event) {
  const target = e.target as HTMLInputElement;
  localValue.value = target.value;
  emit("update:modelValue", target.value);
}

// 清除搜索
function clearSearch() {
  localValue.value = "";
  emit("update:modelValue", "");
  inputRef.value?.focus();
}
</script>

<template>
  <div class="search-box" :class="colorClass">
    <i class="fas fa-search search-icon"></i>
    <input
      ref="inputRef"
      type="text"
      class="search-input"
      :value="localValue"
      :placeholder="placeholder"
      @input="handleInput"
    />
    <button
      v-show="showClear"
      class="search-clear"
      @click="clearSearch"
      title="清除"
    >
      <i class="fas fa-times"></i>
    </button>
  </div>
</template>

<style scoped>
.search-box {
  position: relative;
  display: flex;
  align-items: center;
  flex: 1;
  max-width: 280px;
  min-width: 120px;
}

.search-icon {
  position: absolute;
  left: 16px;
  font-size: 16px;
  pointer-events: none;
  transition: color 0.25s ease;
  color: hsl(var(--text-muted));
}

.search-input {
  width: 100%;
  height: 40px;
  padding: 0 38px 0 42px;
  font-size: 14px;
  border: 1px solid hsl(var(--glass-border));
  border-radius: 12px;
  outline: none;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  background: hsl(var(--glass-bg));
  backdrop-filter: blur(var(--glass-blur));
  -webkit-backdrop-filter: blur(var(--glass-blur));
  color: hsl(var(--text-primary));
  box-shadow: var(--shadow-sm), var(--glass-border-glow);
}

.search-input::placeholder {
  color: hsl(var(--text-muted));
}

.search-input:focus {
  background: hsl(var(--glass-bg));
  border-color: hsl(var(--primary) / 0.5);
  box-shadow: 
    var(--shadow-md),
    0 0 0 2px hsl(var(--primary) / 0.15);
}

.search-input:focus + .search-icon {
  color: hsl(var(--primary));
}

.search-clear {
  position: absolute;
  right: 12px;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: none;
  background: transparent;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.25s ease;
  color: hsl(var(--text-muted));
}

.search-clear:hover {
  color: hsl(var(--primary));
  background: hsl(var(--primary) / 0.1);
}

.search-clear i {
  font-size: 11px;
}



/* 响应式 */
@media (max-width: 640px) {
  .search-box {
    max-width: 160px;
    min-width: 100px;
  }

  .search-input {
    height: 32px;
    font-size: 12px;
    padding: 0 32px;
  }

  .search-icon {
    left: 10px;
    font-size: 11px;
  }

  .search-clear {
    right: 6px;
    width: 20px;
    height: 20px;
  }

  .search-clear i {
    font-size: 10px;
  }
}
</style>
