<template>
  <transition name="modal-fade">
    <div 
      v-if="appState.isLogoutModalOpen" 
      class="modal-backdrop"
      @click.self="actions.closeLogoutModal"
    >
      <div class="modal-card logout-modal-card">
        <!-- Close button -->
        <button class="btn-close-modal" @click="actions.closeLogoutModal" title="取消">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>

        <div class="logout-modal-body">
          <!-- Danger warning icon -->
          <div class="logout-icon-wrapper">
            <svg width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"></path>
              <polyline points="16 17 21 12 16 7"></polyline>
              <line x1="21" y1="12" x2="9" y2="12"></line>
            </svg>
          </div>

          <h3 class="logout-title">退出登录确认</h3>
          <p class="logout-desc">
            确定要退出当前账号「<strong>{{ appState.user.name }}</strong>」吗？
          </p>
          <p class="logout-subdesc">
            退出后将返回「财不外露」官方介绍页。您的会员权益与浏览记录将完好保存，随时可再次登录进入。
          </p>

          <div class="logout-actions-row">
            <button class="btn-cancel" @click="actions.closeLogoutModal">
              取消
            </button>
            <button class="btn-confirm-logout" @click="actions.confirmLogout">
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"></path>
                <polyline points="16 17 21 12 16 7"></polyline>
                <line x1="21" y1="12" x2="9" y2="12"></line>
              </svg>
              确认退出
            </button>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue';
import { appState, actions } from '@/stores/community';

const handleKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Escape' && appState.isLogoutModalOpen) {
    actions.closeLogoutModal();
  }
};

onMounted(() => {
  window.addEventListener('keydown', handleKeydown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown);
});
</script>

<style scoped>
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(15, 23, 42, 0.6);
  backdrop-filter: blur(4px);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.modal-card.logout-modal-card {
  width: 100%;
  max-width: 420px;
  background: #ffffff;
  border-radius: var(--radius-card);
  border: 1px solid var(--border-light);
  box-shadow: none;
  position: relative;
  overflow: hidden;
  padding: 32px 28px 26px;
  text-align: center;
}

.btn-close-modal {
  position: absolute;
  top: 14px;
  right: 14px;
  background: transparent;
  border: none;
  color: var(--text-muted);
  cursor: pointer;
  padding: 6px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all var(--transition-fast);
}

.btn-close-modal:hover {
  background: var(--bg-subtle);
  color: var(--text-primary);
}

.logout-icon-wrapper {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: #fef2f2;
  border: 1px solid #fee2e2;
  color: #ef4444;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 18px;
}

.logout-title {
  font-size: 19px;
  font-weight: 800;
  color: var(--primary-navy);
  margin-bottom: 8px;
}

.logout-desc {
  font-size: 14px;
  color: var(--text-primary);
  line-height: 1.5;
  margin-bottom: 6px;
}

.logout-desc strong {
  color: var(--primary-navy);
}

.logout-subdesc {
  font-size: 13px;
  color: var(--text-muted);
  line-height: 1.5;
  margin-bottom: 24px;
}

.logout-actions-row {
  display: flex;
  gap: 12px;
}

.btn-cancel {
  flex: 1;
  height: 42px;
  border-radius: 8px;
  border: 1px solid var(--border-light);
  background: #ffffff;
  color: var(--text-secondary);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all var(--transition-fast);
}

.btn-cancel:hover {
  background: var(--bg-subtle);
  color: var(--text-primary);
  border-color: #cbd5e1;
}

.btn-confirm-logout {
  flex: 1.2;
  height: 42px;
  border-radius: 8px;
  border: 1px solid #ef4444;
  background: #ef4444;
  color: #ffffff;
  font-size: 14px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  cursor: pointer;
  transition: all var(--transition-fast);
}

.btn-confirm-logout:hover {
  background: #dc2626;
  border-color: #dc2626;
}

.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.2s ease;
}

.modal-fade-enter-from,
.modal-fade-leave-to {
  opacity: 0;
}
</style>
