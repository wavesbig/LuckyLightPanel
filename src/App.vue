<script setup lang="ts">
import { onMounted, computed, watch, ref, provide } from "vue";
import { useNavStore } from "@/stores/nav";
import { useConfigStore } from "@/stores/config";
import type { TabType } from "@/types";
import TechBackground from "@/components/common/TechBackground.vue";
import LoadingScreen from "@/components/common/LoadingScreen.vue";
import AppSidebar from "@/components/layout/AppSidebar.vue";
import SiteGrid from "@/components/sites/SiteGrid.vue";
import DockerGrid from "@/components/docker/DockerGrid.vue";
import ServiceGrid from "@/components/luckyServices/ServiceGrid.vue";
import SettingsPanel from "@/components/settings/SettingsPanel.vue";
import BackToTop from "@/components/common/BackToTop.vue";
import LinkDropdown from "@/components/common/LinkDropdown.vue";
import AppToolbar from "@/components/layout/AppToolbar.vue";
import SearchBar from "@/components/common/SearchBar.vue";

const navStore = useNavStore();
const configStore = useConfigStore();

// 计算内容区域的左边距 (Desktop only)
const contentMargin = computed(() => 
  configStore.sidebarCollapsed ? '136px' : '308px'
);

// 链接下拉菜单组件引用
const linkDropdownRef = ref<InstanceType<typeof LinkDropdown> | null>(null);

// 提供给子组件
provide("linkDropdown", {
  show: (site: any, urls: any[], target: HTMLElement) => {
    linkDropdownRef.value?.show(site, urls, target);
  },
});

// 计算属性
const isLoading = computed(() => navStore.isLoading);
const currentTab = computed(() => configStore.currentTab);

// 各标签页是否有数据
const hasSites = computed(
  () => navStore.sitesEnabled && navStore.allSites.length > 0,
);
const hasDocker = computed(
  () => navStore.dockerEnabled && navStore.allContainers.length > 0,
);
const hasLuckyServices = computed(
  () => navStore.luckyServicesEnabled && navStore.allLuckyServices.length > 0,
);

// 获取第一个可用的标签页
function getFirstAvailableTab(): TabType | null {
  if (hasSites.value) return "sites";
  if (hasDocker.value) return "docker";
  if (hasLuckyServices.value) return "luckyServices";
  return null;
}

// 检查并修正当前标签页
function ensureValidTab() {
  const tab = currentTab.value;
  const isCurrentTabValid =
    (tab === "sites" && hasSites.value) ||
    (tab === "docker" && hasDocker.value) ||
    (tab === "luckyServices" && hasLuckyServices.value);

  if (!isCurrentTabValid) {
    const firstTab = getFirstAvailableTab();
    if (firstTab) {
      configStore.setCurrentTab(firstTab);
    }
  }
}

// 拼接图标 URL
function getIconUrl(path: string) {
  if (!path) return "";
  if (path.startsWith("http://") || path.startsWith("https://")) {
    return path;
  }
  return `./backend/iconlibs/${path}`;
}

// 主题切换
watch(
  () => configStore.effectiveTheme,
  (theme) => {
    document.documentElement.setAttribute("data-theme", theme);
  },
  { immediate: true },
);

// 浏览器标题更新
watch(
  () => navStore.panelTitle,
  (title) => {
    if (title) {
      document.title = title;
    }
  },
  { immediate: true },
);

// 浏览器 Favicon 更新
watch(
  () => navStore.panelFavicon,
  (favicon) => {
    if (favicon) {
      const faviconUrl = getIconUrl(favicon);
      let link = document.querySelector("link[rel~='icon']") as HTMLLinkElement;
      if (!link) {
        link = document.createElement("link");
        link.rel = "icon";
        document.head.appendChild(link);
      }
      link.href = faviconUrl;
    }
  },
  { immediate: true },
);

// 监听标签页变化，控制数据轮询
watch(
  currentTab,
  (newTab: TabType) => {
    navStore.stopDockerStatsPolling();
    navStore.stopLuckyServicesStatsPolling();

    if (newTab === "docker" && navStore.dockerEnabled) {
      navStore.startDockerStatsPolling();
    } else if (newTab === "luckyServices" && navStore.luckyServicesEnabled) {
      navStore.startLuckyServicesStatsPolling();
    }
  },
  { immediate: true },
);

// 初始化
onMounted(async () => {
  // 先加载本地配置（包含默认值）
  configStore.loadConfig();

  // 并行加载数据和服务器配置
  const [, serverConfig] = await Promise.all([
    navStore.loadAllData(),
    navStore.fetchServerConfig(), // 失败时返回 null，不会影响正常运行
  ]);

  // 尝试应用服务器配置（仅在没有本地配置时生效）
  // 如果服务器配置不可用，将使用内置默认配置
  if (serverConfig) {
    configStore.applyServerConfig(serverConfig);
  }

  // 如果当前是自动或混合模式，调用接口重新识别网络类型
  const mode = configStore.networkMode;
  if (mode === "auto" || mode === "hybrid") {
    await navStore.fetchNetworkType();
  }

  // 检查当前标签页是否有效，无效则切换到第一个可用标签页
  ensureValidTab();
  
  // 移动端默认折叠侧边栏
  if (window.innerWidth < 768) {
    configStore.setSidebarCollapsed(true);
  }
});
</script>

<template>
  <!-- 科技感动态背景 -->
  <TechBackground />

  <!-- 加载屏幕 -->
  <LoadingScreen v-if="isLoading" />

  <!-- 主内容 -->
  <div v-else class="app-main">
    <!-- 侧边栏 -->
    <AppSidebar />
    
    <!-- 移动端遮罩层 -->
    <div 
      class="mobile-overlay" 
      :class="{ show: !configStore.sidebarCollapsed }"
      @click="configStore.setSidebarCollapsed(true)"
    />

    <!-- 主区域 -->
    <main class="main-content" :style="{ marginLeft: contentMargin }">
      <!-- 顶部头部：汉堡按钮和搜索栏 -->
      <header class="app-header">
        <!-- 移动端汉堡按钮 -->
        <button 
          class="hamburger-btn" 
          @click="configStore.toggleSidebar()"
          title="展开侧边栏"
        >
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="3" y1="12" x2="21" y2="12"></line><line x1="3" y1="6" x2="21" y2="6"></line><line x1="3" y1="18" x2="21" y2="18"></line></svg>
        </button>

        <!-- 居中的搜索栏 -->
        <div v-if="configStore.showSearch" class="search-wrapper">
          <SearchBar />
        </div>
      </header>

      <!-- 视图操作栏 -->
      <div class="view-toolbar">
        <AppToolbar />
      </div>

      <!-- 内容区域 -->
      <div class="content-area">
        <!-- 站点网格 -->
        <SiteGrid v-if="currentTab === 'sites' && hasSites" />

        <!-- Docker 网格 -->
        <DockerGrid v-else-if="currentTab === 'docker' && hasDocker" />

        <!-- Lucky 服务网格 -->
        <ServiceGrid
          v-else-if="currentTab === 'luckyServices' && hasLuckyServices"
        />
      </div>
    </main>

    <!-- 设置面板 -->
    <SettingsPanel />

    <!-- 返回顶部 -->
    <BackToTop />

    <!-- 链接选择下拉菜单 -->
    <LinkDropdown ref="linkDropdownRef" />
  </div>
</template>

<style scoped>
.app-main {
  min-height: 100vh;
  min-height: 100dvh;
  display: flex; /* Flex row by default */
}

.main-content {
  flex: 1;
  /* 
     Floating Sidebar Logic:
     Margin handled by :style binding in template for better reliability.
  */
  padding: 1.5rem 2rem;
  transition: margin-left 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
  display: flex;
  flex-direction: column;
}

/* 移动端遮罩层 */
.mobile-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(4px);
  z-index: 900; /* Sidebar is 1000 */
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.3s ease;
  display: none;
}

/* 顶部搜索区 */
.app-header {
  display: flex;
  align-items: center;
  justify-content: center; /* 默认居中对齐 */
  position: relative;
  min-height: 56px;
  margin-bottom: 2rem;
}

/* 搜索栏包装器 */
.search-wrapper {
  width: 100%;
  max-width: 720px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

/* 视图工具栏 */
.view-toolbar {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 1.5rem;
  z-index: 10;
  position: relative;
}

/* 汉堡按钮 - 默认隐藏，脱离文档流以防挤占完美居中空间 */
.hamburger-btn {
  display: none;
  position: absolute;
  left: 0;
  background: transparent;
  border: none;
  color: hsl(var(--text-primary));
  cursor: pointer;
  padding: 0.5rem;
  border-radius: var(--radius-md);
  transition: background-color 0.2s;
}

.hamburger-btn:hover {
  background: hsl(var(--bg-elevated));
}

/* 响应式调整 */
@media (max-width: 1024px) {
  .app-header {
    margin-bottom: 1.5rem;
  }
}

@media (max-width: 768px) {
  .main-content {
    margin-left: 0 !important; /* 强制覆盖 v-bind */
    padding: 1rem;
  }
  
  .mobile-overlay {
    display: block;
  }
  
  .mobile-overlay.show {
    opacity: 1;
    pointer-events: auto;
  }
  
  .app-header {
    justify-content: flex-start; /* 移动端靠左为汉堡留位 */
    margin-bottom: 1.5rem;
  }

  .hamburger-btn {
    display: flex;
    position: relative; /* 恢复文档流 */
    margin-right: 0.75rem;
  }
  
  .search-wrapper {
    flex: 1;
    max-width: none;
  }
  
  .view-toolbar {
    justify-content: flex-start; /* 小屏工具栏可左对齐 */
  }
}
</style>
