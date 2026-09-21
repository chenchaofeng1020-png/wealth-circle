<template>
  <div v-if="appState.isMembersModalOpen" class="members-modal-backdrop" @click.self="actions.closeMembersModal">
    <div class="members-modal-card animate-scale-up" role="dialog" aria-modal="true">
      <!-- Modal Header -->
      <div class="modal-header">
        <div class="header-left">
          <div class="header-icon-wrap">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
              <circle cx="9" cy="7" r="4"></circle>
              <path d="M23 21v-2a4 4 0 0 0-3-3.87"></path>
              <path d="M16 3.13a4 4 0 0 1 0 7.75"></path>
            </svg>
          </div>
          <div class="header-title-group">
            <div class="title-row">
              <h3 class="modal-title">社区全部成员</h3>
              <span class="member-total-chip">14,280 位圈友</span>
            </div>
            <p class="modal-subtitle">严选全球资产配置实战派同侪，汇聚资深投研老兵与高净值成长者</p>
          </div>
        </div>

        <button class="modal-close-btn" @click="actions.closeMembersModal" title="关闭弹窗 (Esc)">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2">
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>
      </div>

      <!-- Search & Category Filters -->
      <div class="modal-toolbar">
        <div class="search-box">
          <svg class="search-icon" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="11" cy="11" r="8"></circle>
            <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
          </svg>
          <input 
            type="text" 
            v-model="searchKeyword" 
            placeholder="搜索成员姓名、头衔、投资偏好或标签..." 
            class="search-input"
          />
          <button v-if="searchKeyword" class="clear-search-btn" @click="searchKeyword = ''">✕</button>
        </div>

        <div class="role-tabs">
          <button 
            v-for="tab in filterTabs" 
            :key="tab.key" 
            :class="['role-tab-btn', { active: currentTab === tab.key }]"
            @click="currentTab = tab.key"
          >
            {{ tab.label }}
            <span class="tab-count">{{ tab.count }}</span>
          </button>
        </div>
      </div>

      <!-- Members List Container -->
      <div class="members-scroll-body">
        <div v-if="filteredMembers.length" class="members-grid">
          <div 
            v-for="m in filteredMembers" 
            :key="m.id" 
            class="member-item-card"
          >
            <!-- Avatar & Role Badge -->
            <div class="item-avatar-wrap">
              <img :src="m.avatar" class="member-avatar" :alt="m.name" />
              <span v-if="m.role === 'founder'" class="role-dot dot-founder" title="星主/合伙人">👑</span>
              <span v-else-if="m.role === 'assistant'" class="role-dot dot-assistant" title="特邀助教/导师">🎓</span>
              <span v-else-if="m.role === 'vip'" class="role-dot dot-vip" title="VIP会员">⭐️</span>
            </div>

            <!-- Member Main Brief Info -->
            <div class="item-info">
              <div class="info-top-row">
                <span class="member-name">{{ m.name }}</span>
                <span :class="['role-badge', `badge-${m.role}`]">
                  {{ getRoleLabel(m.role) }}
                </span>
                <span class="joined-days-text">已入圈 {{ getJoinedDays(m.joinedAt) }}</span>
              </div>

              <div class="member-headline" :title="m.roleTitle || m.notes || ''">
                {{ m.roleTitle || m.notes || '专注稳健资产配置' }}
              </div>

              <div class="member-bio-text" :title="m.bio || ''">
                {{ m.bio || '坚持长钱长投，与优秀企业与时间做朋友。' }}
              </div>
            </div>

            <!-- Action Button -->
            <div class="item-action-wrap">
              <button 
                v-if="m.role === 'founder' || m.role === 'assistant'" 
                class="btn-action-primary"
                @click="handleAskMember(m)"
              >
                向TA提问
              </button>
              <button 
                v-else 
                :class="['btn-action-subtle', { followed: followedIds.has(m.id) }]"
                @click="toggleFollow(m)"
              >
                {{ followedIds.has(m.id) ? '已关注' : '+ 关注' }}
              </button>
            </div>
          </div>
        </div>

        <!-- Empty State -->
        <div v-else class="empty-search-state">
          <div class="empty-icon">🔍</div>
          <div class="empty-title">未找到匹配的社区成员</div>
          <p class="empty-desc">尝试更换搜索关键词，或切换上方筛选标签</p>
          <button class="btn-reset-search" @click="resetFilters">清空搜索条件</button>
        </div>
      </div>

      <!-- Modal Footer -->
      <div class="modal-footer">
        <div class="footer-hint">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="12" cy="12" r="10"></circle>
            <line x1="12" y1="16" x2="12" y2="12"></line>
            <line x1="12" y1="8" x2="12.01" y2="8"></line>
          </svg>
          <span>社区成员实名合规审核，数据实时更新 · 严禁群发广告与私自荐股</span>
        </div>
        <button class="btn-close-footer" @click="actions.closeMembersModal">
          关闭
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { appState, actions } from '@/stores/community';
import { MemberRecord, UserRole } from '@/types';

const searchKeyword = ref('');
const currentTab = ref<'all' | 'founder' | 'assistant' | 'vip' | 'guest'>('all');
const followedIds = ref<Set<string>>(new Set(['m-3', 'm-10']));

const filterTabs = computed(() => {
  const members = appState.members;
  return [
    { key: 'all' as const, label: '全部圈友', count: members.length },
    { key: 'founder' as const, label: '星主/合伙人', count: members.filter(m => m.role === 'founder').length },
    { key: 'assistant' as const, label: '特邀专家/助教', count: members.filter(m => m.role === 'assistant').length },
    { key: 'vip' as const, label: 'VIP白金会员', count: members.filter(m => m.role === 'vip').length },
    { key: 'guest' as const, label: '新晋圈友', count: members.filter(m => m.role === 'guest').length },
  ];
});

const filteredMembers = computed(() => {
  let list = appState.members;

  // Filter by role tab
  if (currentTab.value !== 'all') {
    list = list.filter(m => m.role === currentTab.value);
  }

  // Filter by search keyword
  if (searchKeyword.value.trim()) {
    const q = searchKeyword.value.trim().toLowerCase();
    list = list.filter(m => {
      const nameMatch = m.name.toLowerCase().includes(q);
      const titleMatch = (m.roleTitle || '').toLowerCase().includes(q);
      const bioMatch = (m.bio || '').toLowerCase().includes(q);
      const notesMatch = (m.notes || '').toLowerCase().includes(q);
      return nameMatch || titleMatch || bioMatch || notesMatch;
    });
  }

  return list;
});

const getRoleLabel = (role: UserRole) => {
  switch (role) {
    case 'founder': return '圈主/星主';
    case 'admin': return '管理员';
    case 'assistant': return '特邀助教/导师';
    case 'vip': return 'VIP会员';
    default: return '认证圈友';
  }
};

const getJoinedDays = (joinedAt: string) => {
  if (!joinedAt) return '1 天';
  const start = new Date(joinedAt).getTime();
  const now = new Date('2024-09-18').getTime();
  const diffDays = Math.max(1, Math.floor((now - start) / (1000 * 60 * 60 * 24)));
  return `${diffDays} 天`;
};

const toggleFollow = (m: MemberRecord) => {
  if (followedIds.value.has(m.id)) {
    followedIds.value.delete(m.id);
    actions.showToast(`已取消关注 ${m.name}`);
  } else {
    followedIds.value.add(m.id);
    actions.showToast(`已关注圈友 ${m.name}，TA的最新动态将优先推送`);
  }
};

const handleAskMember = (m: MemberRecord) => {
  actions.closeMembersModal();
  actions.showToast(`已向导师 ${m.name} 发起咨询通道，请在右侧向TA提问`);
};

const resetFilters = () => {
  searchKeyword.value = '';
  currentTab.value = 'all';
};

// Handle ESC key to close modal
const onKeyDown = (e: KeyboardEvent) => {
  if (e.key === 'Escape' && appState.isMembersModalOpen) {
    actions.closeMembersModal();
  }
};

onMounted(() => {
  window.addEventListener('keydown', onKeyDown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', onKeyDown);
});
</script>

<style scoped>
.members-modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(15, 23, 42, 0.6);
  backdrop-filter: blur(4px);
  z-index: 1050;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
}

.members-modal-card {
  width: 100%;
  max-width: 780px;
  background: #ffffff;
  border-radius: var(--radius-card);
  border: 1px solid var(--border-light);
  box-shadow: 0 20px 25px -5px rgba(15, 23, 42, 0.15), 0 8px 10px -6px rgba(15, 23, 42, 0.1);
  display: flex;
  flex-direction: column;
  max-height: 85vh;
  overflow: hidden;
}

/* Header */
.modal-header {
  padding: 20px 24px 16px 24px;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  border-bottom: 1px solid var(--border-light);
}

.header-left {
  display: flex;
  align-items: center;
  gap: 14px;
}

.header-icon-wrap {
  width: 44px;
  height: 44px;
  border-radius: 10px;
  background: var(--brand-green-light);
  color: var(--brand-green-hover);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.header-title-group {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.title-row {
  display: flex;
  align-items: center;
  gap: 10px;
}

.modal-title {
  font-size: 18px;
  font-weight: 800;
  color: var(--primary-navy);
  margin: 0;
}

.member-total-chip {
  font-size: 11px;
  font-weight: 700;
  color: var(--brand-green-hover);
  background: var(--brand-green-light);
  border: 1px solid var(--brand-green-border);
  padding: 1px 7px;
  border-radius: 20px;
}

.modal-subtitle {
  font-size: 12.5px;
  color: #64748b;
  margin: 0;
}

.modal-close-btn {
  background: transparent;
  border: none;
  color: #94a3b8;
  padding: 6px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.15s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-close-btn:hover {
  background: #f1f5f9;
  color: var(--primary-navy);
}

/* Toolbar */
.modal-toolbar {
  padding: 14px 24px;
  background: #f8fafc;
  border-bottom: 1px solid var(--border-light);
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.search-box {
  position: relative;
  width: 100%;
}

.search-icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: #94a3b8;
  pointer-events: none;
}

.search-input {
  width: 100%;
  padding: 8px 34px 8px 36px;
  background: #ffffff;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  font-size: 13px;
  color: var(--primary-navy);
  outline: none;
  transition: border-color 0.15s;
}

.search-input:focus {
  border-color: var(--brand-green);
}

.clear-search-btn {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  background: transparent;
  border: none;
  color: #94a3b8;
  cursor: pointer;
  font-size: 12px;
  padding: 4px;
}

.clear-search-btn:hover {
  color: #475569;
}

.role-tabs {
  display: flex;
  align-items: center;
  gap: 8px;
  overflow-x: auto;
  padding-bottom: 2px;
}

.role-tab-btn {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  color: #475569;
  font-size: 12.5px;
  font-weight: 600;
  padding: 5px 12px;
  border-radius: 20px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 6px;
  white-space: nowrap;
  transition: all 0.15s;
}

.role-tab-btn:hover {
  background: #f1f5f9;
  color: var(--primary-navy);
}

.role-tab-btn.active {
  background: var(--primary-navy);
  border-color: var(--primary-navy);
  color: #ffffff;
}

.role-tab-btn.active .tab-count {
  background: rgba(255, 255, 255, 0.2);
  color: #ffffff;
}

.tab-count {
  font-size: 11px;
  font-weight: 700;
  background: #f1f5f9;
  color: #64748b;
  padding: 1px 6px;
  border-radius: 10px;
}

/* Scroll Body */
.members-scroll-body {
  flex: 1;
  overflow-y: auto;
  padding: 16px 24px;
}

.members-grid {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.member-item-card {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 12px 14px;
  border: 1px solid #eaedf1;
  border-radius: var(--radius-md);
  background: #ffffff;
  transition: all 0.15s;
}

.member-item-card:hover {
  border-color: #cbd5e1;
  background: #fafbfc;
}

/* Avatar Wrap */
.item-avatar-wrap {
  position: relative;
  width: 46px;
  height: 46px;
  flex-shrink: 0;
}

.member-avatar {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  object-fit: cover;
  border: 1.5px solid #e2e8f0;
}

.role-dot {
  position: absolute;
  right: -2px;
  bottom: -2px;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 10px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.15);
}

/* Member Info */
.item-info {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.info-top-row {
  display: flex;
  align-items: center;
  gap: 8px;
}

.member-name {
  font-size: 14px;
  font-weight: 700;
  color: var(--primary-navy);
}

.role-badge {
  font-size: 11px;
  font-weight: 600;
  padding: 1px 6px;
  border-radius: 4px;
}

.badge-founder {
  background: #fef3c7;
  color: #b45309;
  border: 1px solid #fde68a;
}

.badge-assistant {
  background: #e0f2fe;
  color: #0369a1;
  border: 1px solid #bae6fd;
}

.badge-vip {
  background: #ecfdf5;
  color: #059669;
  border: 1px solid #a7f3d0;
}

.badge-guest {
  background: #f1f5f9;
  color: #64748b;
  border: 1px solid #e2e8f0;
}

.joined-days-text {
  font-size: 11.5px;
  color: #94a3b8;
  margin-left: auto;
}

.member-headline {
  font-size: 12.5px;
  font-weight: 600;
  color: #334155;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.member-bio-text {
  font-size: 12px;
  color: #64748b;
  line-height: 1.4;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Actions */
.item-action-wrap {
  flex-shrink: 0;
}

.btn-action-primary {
  background: var(--brand-green);
  color: #ffffff;
  border: none;
  font-size: 12px;
  font-weight: 700;
  padding: 6px 12px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.15s;
}

.btn-action-primary:hover {
  background: var(--brand-green-hover);
}

.btn-action-subtle {
  background: #f1f5f9;
  color: #475569;
  border: 1px solid #e2e8f0;
  font-size: 12px;
  font-weight: 600;
  padding: 5px 12px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.15s;
}

.btn-action-subtle:hover {
  background: #e2e8f0;
  color: var(--primary-navy);
}

.btn-action-subtle.followed {
  background: #ecfdf5;
  color: #059669;
  border-color: #a7f3d0;
}

/* Empty State */
.empty-search-state {
  padding: 48px 16px;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.empty-icon {
  font-size: 36px;
  margin-bottom: 8px;
}

.empty-title {
  font-size: 15px;
  font-weight: 700;
  color: var(--primary-navy);
}

.empty-desc {
  font-size: 12.5px;
  color: #64748b;
  margin: 6px 0 16px 0;
}

.btn-reset-search {
  background: #f1f5f9;
  border: 1px solid #cbd5e1;
  color: #334155;
  font-size: 12.5px;
  padding: 6px 14px;
  border-radius: 6px;
  cursor: pointer;
}

/* Footer */
.modal-footer {
  padding: 14px 24px;
  background: #fafbfc;
  border-top: 1px solid var(--border-light);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.footer-hint {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  color: #94a3b8;
}

.btn-close-footer {
  background: #ffffff;
  border: 1px solid #cbd5e1;
  color: #475569;
  font-size: 13px;
  font-weight: 600;
  padding: 6px 16px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.15s;
}

.btn-close-footer:hover {
  background: #f1f5f9;
  color: var(--primary-navy);
}
</style>
