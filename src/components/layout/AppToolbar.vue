<script setup lang="ts">
import { computed } from "vue";
import { storeToRefs } from "pinia";
import { useNavStore } from "@/stores/nav";
import { useConfigStore } from "@/stores/config";
import SearchBox from "@/components/common/SearchBox.vue";
import GroupDropdown from "@/components/common/GroupDropdown.vue";
import NetworkModeDropdown from "@/components/common/NetworkModeDropdown.vue";
import LayoutSwitcher from "@/components/common/LayoutSwitcher.vue";
import DockerLayoutSwitcher from "@/components/docker/DockerLayoutSwitcher.vue";
import ServiceLayoutSwitcher from "@/components/luckyServices/ServiceLayoutSwitcher.vue";
import DockerOverview from "@/components/docker/DockerOverview.vue";
import type { Site, DockerContainer, LuckyService, Group } from "@/types";

const navStore = useNavStore();
const configStore = useConfigStore();
const { currentTab, searchKeywords } = storeToRefs(configStore);

// 当前标签页颜色主题
const currentColor = computed(() => {
  switch (currentTab.value) {
    case "docker":
      return "docker";
    case "luckyServices":
      return "green";
    default:
      return "cyan";
  }
});

// 搜索关键字处理
const searchKeyword = computed({
  get: () => {
    switch (currentTab.value) {
      case "sites":
        return searchKeywords.value.sites;
      case "docker":
        return searchKeywords.value.docker;
      case "luckyServices":
        return searchKeywords.value.luckyServices;
      default:
        return "";
    }
  },
  set: (val: string) => {
    switch (currentTab.value) {
      case "sites":
        searchKeywords.value.sites = val;
        break;
      case "docker":
        searchKeywords.value.docker = val;
        break;
      case "luckyServices":
        searchKeywords.value.luckyServices = val;
        break;
    }
  },
});

// 搜索占位符
const searchPlaceholder = computed(() => {
  switch (currentTab.value) {
    case "sites":
      return "搜索站点...";
    case "docker":
      return "搜索容器...";
    case "luckyServices":
      return "搜索服务...";
    default:
      return "搜索...";
  }
});

// 分组列表处理
const groups = computed(() => {
  let allItems: any[] = [];
  let allGroups: Group[] = [];

  switch (currentTab.value) {
    case "sites":
      allItems = navStore.allSites;
      allGroups = navStore.siteGroups;
      break;
    case "docker":
      allItems = navStore.allContainers;
      allGroups = navStore.dockerGroups;
      break;
    case "luckyServices":
      allItems = navStore.allLuckyServices;
      allGroups = navStore.luckyServicesGroups;
      break;
    default:
      return [];
  }

  const groupsWithCount = allGroups
    .map((g: Group) => {
      const count = allItems.filter(
        (item: any) => item.groupKey === g.key,
      ).length;
      return { ...g, count };
    })
    .filter((g) => g.count > 0);

  return [
    { key: "all", name: "全部", icon: "", count: allItems.length },
    ...groupsWithCount,
  ];
});
</script>

<template>
  <div class="app-toolbar">
    <!-- 搜索框 -->
    <SearchBox
      v-model="searchKeyword"
      :placeholder="searchPlaceholder"
      :color="currentColor"
      class="toolbar-search"
    />

    <!-- 右侧操作区 -->
    <div class="toolbar-actions">
      <!-- 分组下拉 -->
      <GroupDropdown
        :groups="groups"
        :current="configStore.currentGroup"
        :color="currentColor"
        @change="configStore.setCurrentGroup"
        @toggle="configStore.toggleGroup"
      />

      <!-- 站点特有控件 -->
      <template v-if="currentTab === 'sites'">
        <NetworkModeDropdown />
        <LayoutSwitcher />
      </template>

      <!-- Docker 特有控件 -->
      <template v-else-if="currentTab === 'docker'">
        <DockerOverview />
        <DockerLayoutSwitcher />
      </template>

      <!-- Lucky 服务特有控件 -->
      <template v-else-if="currentTab === 'luckyServices'">
        <ServiceLayoutSwitcher />
      </template>
    </div>
  </div>
</template>

<style scoped>
.app-toolbar {
  display: flex;
  align-items: center;
  justify-content: flex-end; /* 靠右对齐 */
  gap: 0.75rem;
  flex-wrap: nowrap; /* 禁止换行，避免挤压 */
}

.toolbar-search {
  min-width: 200px;
  flex-shrink: 0; /* 防止被压缩 */
}

.toolbar-actions {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

/* 响应式调整 */
@media (max-width: 768px) {
  .app-toolbar {
    width: 100%;
    justify-content: space-between;
  }

  .toolbar-search {
    flex: 1;
    min-width: 150px;
  }
}

@media (max-width: 480px) {
  .app-toolbar {
    flex-direction: column;
    align-items: stretch;
    gap: 0.5rem;
  }

  .toolbar-actions {
    justify-content: space-between;
    width: 100%;
  }

  /* 在移动端让搜索框占满宽度 */
  .toolbar-search {
    width: 100%;
  }
}
</style>
