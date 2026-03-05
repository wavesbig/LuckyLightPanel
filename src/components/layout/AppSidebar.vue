<script setup lang="ts">
import { useConfigStore } from '@/stores/config'
import { 
  Globe, 
  Container, 
  Settings, 
  Sun, 
  Moon, 
  ChevronLeft, 
  ChevronRight 
} from 'lucide-vue-next'
import { storeToRefs } from 'pinia'
import { Button } from '@/components/ui/button'
import { Tooltip, TooltipContent, TooltipProvider, TooltipTrigger } from '@/components/ui/tooltip'

const configStore = useConfigStore()
const { currentTab, theme, settingsPanelOpen } = storeToRefs(configStore)

const navItems = [
  { id: 'sites', label: '我的站点', icon: Globe },
  { id: 'docker', label: 'Docker 容器', icon: Container }
]

function switchTab(id: any) {
  configStore.updateConfig('currentTab', id)
}

function toggleTheme() {
  const newTheme = theme.value === 'dark' ? 'light' : 'dark'
  configStore.updateConfig('theme', newTheme)
}
</script>

<template>
  <TooltipProvider :delay-duration="200">
    <aside 
      class="app-sidebar glass-panel" 
      :class="{ 'is-collapsed': configStore.sidebarCollapsed }"
    >
      <!-- 顶部 Logo 区域 -->
      <div class="sidebar-header">
        <div class="logo-wrapper">
          <div class="logo-icon">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="w-6 h-6"><path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5"/></svg>
          </div>
          <span class="logo-text" v-show="!configStore.sidebarCollapsed">LightPanel</span>
        </div>
      </div>

      <!-- 导航菜单 -->
      <nav class="sidebar-nav">
        <div 
          v-for="item in navItems" 
          :key="item.id"
          class="nav-item"
          :class="{ active: currentTab === item.id }"
          @click="switchTab(item.id)"
        >
          <div class="nav-icon">
            <component :is="item.icon" :size="20" />
          </div>
          <span class="nav-label" v-show="!configStore.sidebarCollapsed">{{ item.label }}</span>
        </div>
      </nav>

      <!-- 底部操作区 -->
      <div class="sidebar-footer">
        <div class="footer-actions">
          <!-- 主题切换 -->
          <Tooltip>
            <TooltipTrigger as-child>
              <Button
                variant="ghost"
                size="icon"
                class="action-btn"
                :title="theme === 'dark' ? '切换亮色' : '切换暗色'"
                @click="toggleTheme"
              >
                <Moon v-if="theme === 'dark'" :size="18" />
                <Sun v-else :size="18" />
              </Button>
            </TooltipTrigger>
            <TooltipContent side="right">
              {{ theme === 'dark' ? '切换亮色' : '切换暗色' }}
            </TooltipContent>
          </Tooltip>

          <!-- 设置按钮 -->
          <Tooltip>
            <TooltipTrigger as-child>
              <Button
                variant="ghost"
                size="icon"
                class="action-btn"
                :class="{ active: settingsPanelOpen }"
                title="设置"
                @click="configStore.toggleSettingsPanel()"
              >
                <Settings :size="18" />
              </Button>
            </TooltipTrigger>
            <TooltipContent side="right">设置</TooltipContent>
          </Tooltip>

          <!-- 折叠按钮 -->
          <Tooltip>
            <TooltipTrigger as-child>
              <Button
                variant="ghost"
                size="icon"
                class="action-btn collapse-btn"
                :title="configStore.sidebarCollapsed ? '展开' : '折叠'"
                @click="configStore.toggleSidebar()"
              >
                <ChevronRight v-if="configStore.sidebarCollapsed" :size="18" />
                <ChevronLeft v-else :size="18" />
              </Button>
            </TooltipTrigger>
            <TooltipContent side="right">
              {{ configStore.sidebarCollapsed ? '展开侧边栏' : '折叠侧边栏' }}
            </TooltipContent>
          </Tooltip>
        </div>
      </div>
    </aside>
  </TooltipProvider>
</template>

<style scoped>
.app-sidebar {
  position: fixed;
  top: var(--card-gap);
  left: var(--card-gap);
  bottom: var(--card-gap);
  width: var(--sidebar-width);
  background: hsl(var(--glass-bg));
  backdrop-filter: blur(var(--glass-blur));
  -webkit-backdrop-filter: blur(var(--glass-blur));
  border: 1px solid hsl(var(--glass-border));
  box-shadow: var(--shadow-xl), var(--glass-border-glow);
  border-radius: var(--radius-xl);
  z-index: 1000;
  display: flex;
  flex-direction: column;
  transition: all 0.4s cubic-bezier(0.34, 1.56, 0.64, 1); /* Spring-like transition */
  overflow: hidden;
  padding: 1.5rem 0;
}

/* 折叠状态 - Mini Sidebar */
.app-sidebar.is-collapsed {
  width: var(--sidebar-collapsed-width);
}

/* Header */
.sidebar-header {
  padding: 0 1.5rem;
  margin-bottom: 2rem;
  height: 48px;
  display: flex;
  align-items: center;
}

.logo-wrapper {
  display: flex;
  align-items: center;
  gap: 1rem;
  color: hsl(var(--primary));
  font-weight: 700;
  font-size: 1.25rem;
  overflow: hidden;
  white-space: nowrap;
}

.logo-icon {
  min-width: 32px;
  height: 32px;
  background: linear-gradient(135deg, hsl(var(--primary)), hsl(var(--accent)));
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  box-shadow: 0 4px 10px -2px hsl(var(--primary) / 0.4);
}

.logo-text {
  font-family: var(--font-sans);
  background: linear-gradient(to right, hsl(var(--text-primary)), hsl(var(--text-secondary)));
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;
  opacity: 1;
  transition: opacity 0.2s;
}

/* Nav */
.sidebar-nav {
  flex: 1;
  padding: 0 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.nav-item {
  position: relative;
  display: flex;
  align-items: center;
  height: 48px;
  padding: 0 1rem;
  border-radius: var(--radius-lg);
  color: hsl(var(--text-secondary));
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  overflow: hidden;
  white-space: nowrap;
  font-weight: 500;
}

.is-collapsed .nav-item {
  padding: 0;
  justify-content: center;
}

.nav-item:hover {
  background: hsl(var(--primary) / 0.08);
  color: hsl(var(--text-primary));
}

.is-collapsed .nav-item:hover {
  background: hsl(var(--primary) / 0.08);
}

.nav-item.active {
  background: hsl(var(--primary) / 0.15);
  color: hsl(var(--primary));
  font-weight: 600;
  box-shadow: inset 0 0 0 1px hsl(var(--primary) / 0.25), 0 1px 2px hsl(var(--primary) / 0.05);
}

.nav-icon {
  min-width: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.2s ease;
}

.nav-item:hover .nav-icon {
  transform: scale(1.1);
}

.nav-label {
  margin-left: 1rem;
  font-size: 0.95rem;
}

/* Footer */
.sidebar-footer {
  padding: 0 0.75rem;
  margin-top: auto;
}

.footer-actions {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  background: hsl(var(--bg-elevated) / 0.4);
  border: 1px solid hsl(var(--glass-border));
  border-radius: var(--radius-xl);
  padding: 0.25rem;
  gap: 0.25rem;
  box-shadow: inset 0 1px 2px hsl(0 0% 100% / 0.05);
}

.is-collapsed .sidebar-footer {
  padding: 0 0.5rem;
}

.is-collapsed .footer-actions {
  flex-direction: column;
  padding: 0.5rem 0.25rem;
  border-radius: 24px;
}

.action-btn {
  flex: 1;
  height: 36px;
  min-width: 36px;
  border-radius: calc(var(--radius-xl) - 4px);
  display: flex;
  align-items: center;
  justify-content: center;
  background: transparent;
  border: 1px solid transparent;
  color: hsl(var(--text-secondary));
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}

.is-collapsed .action-btn {
  flex: none;
  width: 36px;
}

.action-btn:hover {
  background: hsl(var(--bg-elevated));
  color: hsl(var(--text-primary));
  box-shadow: var(--shadow-sm);
}

.action-btn.active {
  background: hsl(var(--primary) / 0.15);
  color: hsl(var(--primary));
  box-shadow: inset 0 0 0 1px hsl(var(--primary) / 0.2);
}

.collapse-btn {
  color: hsl(var(--text-muted));
}

.collapse-btn:hover {
  color: hsl(var(--primary));
}

/* 响应式调整 */
@media (max-width: 768px) {
  .app-sidebar {
    top: 0;
    left: 0;
    bottom: 0;
    width: var(--sidebar-width);
    border-radius: 0;
    border-left: none;
    border-top: none;
    border-bottom: none;
    transform: translateX(-105%);
    transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  }

  .app-sidebar.is-collapsed {
    /* Mobile collapsed = hidden */
    transform: translateX(-105%); 
    width: var(--sidebar-width); 
  }
  
  /* When expanded on mobile (sidebarCollapsed is false) */
  .app-sidebar:not(.is-collapsed) {
     transform: translateX(0);
  }
}
</style>
