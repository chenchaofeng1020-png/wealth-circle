<template>
  <header class="top-header">
    <!-- Left: Breadcrumb / Active view title -->
    <div class="header-left">
      <div class="view-indicator">
        <span class="view-title">{{ viewTitle }}</span>
      </div>
    </div>

    <!-- Center: Global Search -->
    <div v-if="appState.currentView !== 'admin_members'" class="header-center">
      <div class="search-box">
        <svg class="search-icon" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="11" cy="11" r="8"></circle>
          <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
        </svg>
        <input 
          type="text" 
          placeholder="搜索主题、专栏..." 
          v-model="searchQuery"
          @keyup.enter="handleSearch"
        />
        <span class="search-shortcut">⌘K</span>
      </div>
    </div>

    <!-- Right: Quick actions & identity switch -->
    <div class="header-right">
      <!-- Quick VIP Upgrade / Status (shown when in VIP or Guest mode) -->
      <button 
        v-if="appState.currentPerspective !== 'founder'" 
        class="btn-header-vip"
        @click="appState.isVipJoinModalOpen = true"
      >
        {{ appState.currentPerspective === 'guest' ? '开通会员' : '会员权益' }}
      </button>

      <!-- Notification bell -->
      <div v-if="appState.currentView !== 'admin_members'" class="header-icon-btn" title="通知消息" @click="actions.showToast('暂无未读系统通知')">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"></path>
          <path d="M13.73 21a2 2 0 0 1-3.46 0"></path>
        </svg>
        <span class="unread-dot"></span>
      </div>
    </div>
  </header>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { appState, actions } from '@/stores/community';

const searchQuery = ref('');

const viewTitle = computed(() => {
  switch (appState.currentView) {
    case 'feed': return '首页';
    case 'column': return '专栏';
    case 'course': return '课程';
    case 'live': return '直播';
    case 'user': return '个人中心';
    case 'admin_column': return '专栏管理';
    case 'admin_live': return '直播管理';
    case 'admin_course': return '课程管理';
    case 'admin_content': return '内容管理';
    case 'admin_members': return '成员与权限';
    default: return '首页';
  }
});

const handleSearch = () => {
  if (searchQuery.value.trim()) {
    actions.showToast(`已搜索关键词: "${searchQuery.value.trim()}"`);
  }
};
</script>

<style scoped>
.top-header {
  height: 64px;
  background: #ffffff;
  border-bottom: 1px solid var(--border-light);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 28px;
  position: sticky;
  top: 0;
  z-index: 15;
  flex-shrink: 0;
}

.header-left {
  display: flex;
  align-items: center;
}

.view-indicator {
  display: flex;
  flex-direction: column;
}

.view-title {
  font-size: 16px;
  font-weight: 800;
  color: var(--primary-navy);
}

.header-center {
  margin-left: auto;
  margin-right: 20px;
  width: 100%;
  max-width: 280px;
}

.search-box {
  display: flex;
  align-items: center;
  background: var(--bg-subtle);
  border: 1px solid var(--border-light);
  border-radius: var(--radius-full);
  padding: 6px 14px;
  transition: all var(--transition-fast);
}

.search-box:focus-within {
  background: #ffffff;
  border-color: var(--brand-green);
  box-shadow: 0 0 0 3px rgba(16, 185, 129, 0.12);
}

.search-icon {
  color: var(--text-muted);
  margin-right: 8px;
}

.search-box input {
  border: none;
  background: transparent;
  width: 100%;
  font-size: 13px;
  color: var(--text-primary);
  box-shadow: none;
  padding: 0;
}

.search-shortcut {
  font-size: 10px;
  font-weight: 600;
  color: var(--text-muted);
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: 4px;
  padding: 1px 5px;
}

.header-right {
  display: flex;
  align-items: center;
  gap: 14px;
}

.btn-header-vip {
  background: var(--vip-gold-gradient);
  color: #ffffff;
  font-size: 12px;
  font-weight: 700;
  padding: 6px 12px;
  border-radius: var(--radius-full);
  display: flex;
  align-items: center;
  gap: 4px;
  box-shadow: 0 2px 6px rgba(217, 119, 6, 0.25);
}

.header-icon-btn {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--text-secondary);
  position: relative;
  cursor: pointer;
  transition: background var(--transition-fast);
}

.header-icon-btn:hover {
  background: var(--bg-subtle);
  color: var(--primary-navy);
}

.unread-dot {
  width: 6px;
  height: 6px;
  background: var(--danger-red);
  border-radius: 50%;
  position: absolute;
  top: 7px;
  right: 8px;
}
</style>
