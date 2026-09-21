<template>
  <aside class="sidebar-container">
    <!-- Brand Logo Header -->
    <div class="brand-header" @click="actions.setCurrentView('feed')">
      <div class="logo-emblem">
        <span class="emblem-char">财</span>
      </div>
      <div class="brand-text-block">
        <div class="brand-title">财不外露</div>
        <div class="brand-subtitle">财经私享知识社区</div>
      </div>
    </div>

    <!-- Quick Publish Button -->
    <!-- Main Navigation List -->
    <nav class="nav-section">
      <ul class="nav-list">
        <li 
          :class="['nav-item', { active: appState.currentView === 'feed' }]"
          @click="actions.setCurrentView('feed')"
        >
          <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <rect x="3" y="3" width="7" height="7" rx="1.5" />
            <rect x="14" y="3" width="7" height="7" rx="1.5" />
            <rect x="14" y="14" width="7" height="7" rx="1.5" />
            <rect x="3" y="14" width="7" height="7" rx="1.5" />
          </svg>
          <span class="nav-label">首页</span>
        </li>

        <li 
          :class="['nav-item', { active: appState.currentView === 'column' }]"
          @click="actions.setCurrentView('column')"
        >
          <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20" />
            <path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z" />
          </svg>
          <span class="nav-label">专栏</span>
        </li>

        <li 
          :class="['nav-item', { active: appState.currentView === 'live' }]"
          @click="actions.setCurrentView('live')"
        >
          <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M4.9 19.1C1 15.2 1 8.8 4.9 4.9" />
            <path d="M7.8 16.2c-2.3-2.3-2.3-6.1 0-8.5" />
            <circle cx="12" cy="12" r="2" />
            <path d="M16.2 7.8c2.3 2.3 2.3 6.1 0 8.5" />
            <path d="M19.1 4.9C23 8.8 23 15.2 19.1 19.1" />
          </svg>
          <span class="nav-label">直播</span>
        </li>

        <li 
          :class="['nav-item', { active: appState.currentView === 'course' }]"
          @click="actions.setCurrentView('course')"
        >
          <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polygon points="23 7 16 12 23 17 23 7" />
            <rect x="1" y="5" width="15" height="14" rx="2" ry="2" />
          </svg>
          <span class="nav-label">课程</span>
        </li>
      </ul>

      <!-- Management Portal (Admin) -->
      <div v-if="isManager" class="admin-nav-block">
        <div class="nav-group-title">管理端</div>
        <ul class="nav-list">
          <li 
            :class="['nav-item admin-item', { active: appState.currentView === 'admin_column' || appState.currentView === 'admin_content' }]"
            @click="actions.setCurrentView('admin_column')"
          >
            <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20" />
              <path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z" />
            </svg>
            <span class="nav-label">专栏管理</span>
          </li>

          <li 
            :class="['nav-item admin-item', { active: appState.currentView === 'admin_live' }]"
            @click="actions.setCurrentView('admin_live')"
          >
            <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polygon points="23 7 16 12 23 17 23 7" />
              <rect x="1" y="5" width="15" height="14" rx="2" ry="2" />
            </svg>
            <span class="nav-label">直播管理</span>
          </li>

          <li 
            :class="['nav-item admin-item', { active: appState.currentView === 'admin_course' }]"
            @click="actions.setCurrentView('admin_course')"
          >
            <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M22 10v6M2 10l10-5 10 5-10 5z" />
              <path d="M6 12v5c3 3 9 3 12 0v-5" />
            </svg>
            <span class="nav-label">课程管理</span>
          </li>

          <li 
            :class="['nav-item admin-item', { active: appState.currentView === 'admin_members' }]"
            @click="actions.setCurrentView('admin_members')"
          >
            <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2" />
              <circle cx="9" cy="7" r="4" />
              <path d="M23 21v-2a4 4 0 0 0-3-3.87" />
              <path d="M16 3.13a4 4 0 0 1 0 7.75" />
            </svg>
            <span class="nav-label">成员与权限</span>
          </li>
        </ul>
      </div>
    </nav>

    <!-- Bottom User Profile Entrance with Popover Menu -->
    <div class="user-entry-section" ref="userSectionRef">
      <!-- Upward Popover Menu (containing logout and account links) -->
      <transition name="popover-fade">
        <div v-if="isMenuOpen" class="user-popover-menu">
          <!-- Popover Header: User Info -->
          <div class="popover-user-card" @click="goToUserCenter">
            <img :src="appState.user.avatar" class="popover-avatar" alt="Avatar" />
            <div class="popover-meta">
              <div class="popover-name-row">
                <span class="popover-name">{{ appState.user.name }}</span>
                <span v-if="appState.user.role !== 'guest'" :class="['badge', appState.user.role === 'founder' ? 'badge-founder' : 'badge-vip']">
                  {{ appState.user.role === 'founder' ? '圈主' : 'VIP' }}
                </span>
              </div>
              <div class="popover-sub">UID: {{ appState.user.id }} · {{ appState.user.roleTitle }}</div>
            </div>
          </div>

          <div class="popover-divider"></div>

          <!-- Menu items -->
          <div class="popover-menu-list">
            <div class="popover-menu-item" @click="goToUserCenter">
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path>
                <circle cx="12" cy="7" r="4"></circle>
              </svg>
              <span>个人中心与资产</span>
            </div>

            <div class="popover-menu-item" @click="openVipModal">
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon>
              </svg>
              <span>会员特权与续费</span>
            </div>

            <div v-if="appState.currentPerspective !== 'founder'" class="popover-menu-item" @click="handleSwitchPerspective('founder')">
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path>
                <circle cx="12" cy="7" r="4"></circle>
                <polyline points="17 8 20 5 23 8"></polyline>
                <line x1="20" y1="5" x2="20" y2="15"></line>
              </svg>
              <span>切换至圈主/管理员视角</span>
            </div>

            <div v-if="appState.currentPerspective !== 'vip'" class="popover-menu-item" @click="handleSwitchPerspective('vip')">
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon>
              </svg>
              <span>切换至VIP会员视角</span>
            </div>

            <div v-if="appState.currentPerspective !== 'guest'" class="popover-menu-item" @click="handleSwitchPerspective('guest')">
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path>
                <circle cx="12" cy="7" r="4"></circle>
              </svg>
              <span>切换至注册用户（未开通会员）视角</span>
            </div>
          </div>

          <div class="popover-divider"></div>

          <!-- Logout item inside menu -->
          <div class="popover-menu-item logout-item" @click="handleLogout">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"></path>
              <polyline points="16 17 21 12 16 7"></polyline>
              <line x1="21" y1="12" x2="9" y2="12"></line>
            </svg>
            <span>退出登录</span>
          </div>
        </div>
      </transition>

      <!-- Bottom User Card Trigger (no subline, clicking opens menu) -->
      <div 
        :class="['user-entry-card', { active: isMenuOpen || appState.currentView === 'user' }]"
        @click="toggleMenu"
        title="点击展开个人账号菜单"
      >
        <div class="user-entry-left">
          <div class="user-avatar-wrap">
            <img :src="appState.user.avatar" class="user-avatar" alt="Avatar" />
            <span :class="['avatar-status-dot', appState.user.role === 'founder' ? 'dot-founder' : appState.user.role === 'vip' ? 'dot-vip' : 'dot-guest']"></span>
          </div>

          <div class="user-name-box">
            <span class="user-name" :title="appState.user.name">{{ appState.user.name }}</span>
            <span v-if="appState.user.role !== 'guest'" :class="['badge', appState.user.role === 'founder' ? 'badge-founder' : 'badge-vip']">
              {{ appState.user.role === 'founder' ? '圈主' : 'VIP' }}
            </span>
          </div>
        </div>

        <div class="user-entry-right">
          <svg 
            class="entry-chevron-icon" 
            :class="{ 'rotate-up': isMenuOpen }" 
            width="13" 
            height="13" 
            viewBox="0 0 24 24" 
            fill="none" 
            stroke="currentColor" 
            stroke-width="2.5"
          >
            <polyline points="18 15 12 9 6 15"></polyline>
          </svg>
        </div>
      </div>
    </div>
  </aside>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { appState, actions, isManager } from '@/stores/community';

const isMenuOpen = ref(false);
const userSectionRef = ref<HTMLElement | null>(null);

const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value;
};

const goToUserCenter = () => {
  isMenuOpen.value = false;
  actions.switchView('user');
};

const openVipModal = () => {
  isMenuOpen.value = false;
  appState.isVipJoinModalOpen = true;
};

const handleSwitchPerspective = (target: 'founder' | 'vip' | 'guest') => {
  isMenuOpen.value = false;
  actions.switchPerspective(target);
};

const handleLogout = () => {
  isMenuOpen.value = false;
  actions.openLogoutModal();
};

const handleDocClick = (e: MouseEvent) => {
  if (userSectionRef.value && !userSectionRef.value.contains(e.target as Node)) {
    isMenuOpen.value = false;
  }
};

const handleKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Escape' && isMenuOpen.value) {
    isMenuOpen.value = false;
  }
};

onMounted(() => {
  document.addEventListener('click', handleDocClick);
  document.addEventListener('keydown', handleKeydown);
});

onUnmounted(() => {
  document.removeEventListener('click', handleDocClick);
  document.removeEventListener('keydown', handleKeydown);
});
</script>

<style scoped>
.sidebar-container {
  width: 260px;
  background: var(--bg-sidebar);
  border-right: 1px solid #e2e8f0;
  display: flex;
  flex-direction: column;
  height: 100vh;
  position: sticky;
  top: 0;
  user-select: none;
  z-index: 20;
  flex-shrink: 0;
}

.brand-header {
  padding: 24px 20px 16px 20px;
  display: flex;
  align-items: center;
  gap: 12px;
  cursor: pointer;
}

.logo-emblem {
  width: 42px;
  height: 42px;
  background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 12px rgba(15, 23, 42, 0.2);
  border: 1px solid rgba(245, 158, 11, 0.35);
  flex-shrink: 0;
}

.emblem-char {
  color: #f59e0b;
  font-size: 22px;
  font-weight: 800;
  font-family: 'PingFang SC', sans-serif;
  letter-spacing: -1px;
}

.brand-text-block {
  overflow: hidden;
}

.brand-title {
  font-size: 18px;
  font-weight: 800;
  color: var(--primary-navy);
  letter-spacing: 0.5px;
  line-height: 1.2;
}

.brand-subtitle {
  font-size: 11px;
  color: var(--text-muted);
  margin-top: 2px;
  letter-spacing: 0.3px;
}

.publish-entry {
  padding: 8px 16px 16px 16px;
}

.btn-quick-publish {
  width: 100%;
  background: var(--brand-green);
  color: #ffffff;
  padding: 11px 16px;
  border-radius: var(--radius-md);
  font-size: 14px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  box-shadow: 0 4px 12px rgba(16, 185, 129, 0.22);
}
.btn-quick-publish:hover {
  background: var(--brand-green-hover);
  transform: translateY(-1px);
  box-shadow: 0 6px 16px rgba(16, 185, 129, 0.3);
}

.nav-section {
  flex: 1;
  overflow-y: auto;
  padding: 0 12px;
}

.nav-group-title {
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  color: #64748b;
  letter-spacing: 0.8px;
  padding: 12px 12px 6px 12px;
}

.nav-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.nav-item {
  display: flex;
  align-items: center;
  padding: 10px 14px;
  border-radius: var(--radius-card);
  color: var(--text-secondary);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all var(--transition-fast);
}

.nav-item:hover {
  background: var(--bg-sidebar-hover);
  color: var(--text-primary);
}

.nav-item.active {
  background: var(--bg-sidebar-active);
  color: var(--primary-navy);
  font-weight: 700;
  box-shadow: none;
}

.nav-item.active .nav-icon {
  color: var(--brand-green);
}

.admin-item.active {
  background: var(--bg-sidebar-active);
  color: #1d4ed8;
  box-shadow: none;
}

.admin-item.active .nav-icon {
  color: #2563eb;
}

.nav-icon {
  margin-right: 12px;
  flex-shrink: 0;
}

.nav-label {
  flex: 1;
}

.nav-badge {
  font-size: 11px;
  font-weight: 600;
  padding: 1px 7px;
  border-radius: 99px;
  background: #e2e8f0;
  color: var(--text-secondary);
}

.nav-badge.hot {
  background: var(--vip-gold-light);
  color: var(--vip-gold);
}

.nav-dot-live {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--danger-red);
  box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.25);
  animation: pulse 1.5s infinite;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.2); opacity: 0.7; }
}

.admin-nav-block {
  margin-top: 18px;
  padding-top: 14px;
  border-top: 1px dashed #cbd5e1;
}

/* Bottom User Profile Section with Upward Popover */
.user-entry-section {
  position: relative;
  padding: 12px 10px;
  background: var(--bg-sidebar);
  border-top: none;
  user-select: none;
}

/* Bottom User Card Trigger */
.user-entry-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 10px;
  background: #ffffff;
  border: none;
  border-radius: var(--radius-card);
  cursor: pointer;
  transition: all var(--transition-fast);
}

.user-entry-card:hover {
  background: #ffffff;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);
}

.user-entry-card.active {
  background: #f0fdf4;
}

.user-entry-left {
  display: flex;
  align-items: center;
  gap: 10px;
  flex: 1;
  min-width: 0;
}

.user-avatar-wrap {
  position: relative;
  width: 36px;
  height: 36px;
  flex-shrink: 0;
}

.user-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  object-fit: cover;
  border: 1px solid #cbd5e1;
  display: block;
}

.avatar-status-dot {
  position: absolute;
  bottom: -1px;
  right: -1px;
  width: 9px;
  height: 9px;
  border-radius: 50%;
  border: 1.5px solid #ffffff;
}

.avatar-status-dot.dot-vip {
  background: #f59e0b;
}

.avatar-status-dot.dot-founder {
  background: var(--brand-green);
}

.avatar-status-dot.dot-guest {
  background: #94a3b8;
}

.user-name-box {
  display: flex;
  align-items: center;
  gap: 6px;
  flex: 1;
  min-width: 0;
}

.user-name {
  font-size: 13px;
  font-weight: 700;
  color: var(--primary-navy);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 110px;
  line-height: 1.3;
}

.user-entry-right {
  display: flex;
  align-items: center;
  padding-left: 4px;
}

.entry-chevron-icon {
  color: #94a3b8;
  transition: transform var(--transition-fast), color var(--transition-fast);
}

.entry-chevron-icon.rotate-up {
  transform: rotate(180deg);
  color: var(--primary-navy);
}

/* Upward Popover Menu */
.user-popover-menu {
  position: absolute;
  bottom: calc(100% + 8px);
  left: 10px;
  right: 10px;
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: var(--radius-card);
  box-shadow: none;
  z-index: 100;
  padding: 8px 0;
  overflow: hidden;
}

.popover-user-card {
  padding: 10px 14px;
  display: flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
  transition: background var(--transition-fast);
}

.popover-user-card:hover {
  background: var(--bg-subtle);
}

.popover-avatar {
  width: 38px;
  height: 38px;
  border-radius: 50%;
  object-fit: cover;
  border: 1px solid #cbd5e1;
  flex-shrink: 0;
}

.popover-meta {
  flex: 1;
  min-width: 0;
}

.popover-name-row {
  display: flex;
  align-items: center;
  gap: 6px;
}

.popover-name {
  font-size: 13px;
  font-weight: 700;
  color: var(--primary-navy);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.popover-sub {
  font-size: 11px;
  color: #64748b;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-top: 2px;
}

.popover-divider {
  height: 1px;
  background: #f1f5f9;
  margin: 4px 0;
}

.popover-menu-list {
  display: flex;
  flex-direction: column;
}

.popover-menu-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 9px 14px;
  font-size: 13px;
  font-weight: 600;
  color: var(--text-secondary);
  cursor: pointer;
  transition: all var(--transition-fast);
}

.popover-menu-item:hover {
  background: var(--bg-subtle);
  color: var(--primary-navy);
}

.popover-menu-item.logout-item {
  color: #ef4444;
}

.popover-menu-item.logout-item:hover {
  background: #fef2f2;
  color: #dc2626;
}

/* Popover Animation */
.popover-fade-enter-active,
.popover-fade-leave-active {
  transition: opacity 0.15s ease, transform 0.15s ease;
}

.popover-fade-enter-from,
.popover-fade-leave-to {
  opacity: 0;
  transform: translateY(6px);
}
</style>
