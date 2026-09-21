<template>
  <div class="member-manage-wrap animate-fade-in">
    <!-- Top Stats Banner -->
    <div class="stats-overview-grid">
      <div class="stat-card">
        <span class="stat-num num-tabular">{{ appState.members.length }}</span>
        <span class="stat-text">圈子总成员</span>
      </div>
      <div class="stat-card">
        <span class="stat-num num-tabular" style="color: #0369a1;">{{ assistantCount }}</span>
        <span class="stat-text">合伙人团队</span>
      </div>
      <div class="stat-card">
        <span class="stat-num num-tabular" style="color: var(--vip-gold);">{{ vipCount }}</span>
        <span class="stat-text">付费会员</span>
      </div>
      <div class="stat-card">
        <span class="stat-num num-tabular" style="color: var(--brand-green-hover);">¥{{ totalRevenue.toLocaleString() }}</span>
        <span class="stat-text">圈子累计收入</span>
      </div>
    </div>

    <!-- Filter & Search Toolbar -->
    <div class="member-toolbar">
      <div class="search-input-wrap">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="11" cy="11" r="8"></circle>
          <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
        </svg>
        <input 
          type="text" 
          placeholder="按成员昵称、手机号搜索..." 
          v-model="searchQuery" 
        />
      </div>

      <div class="toolbar-right">
        <select v-model="filterRole" class="role-filter-select">
          <option value="all">全部角色 ({{ appState.members.length }})</option>
          <option value="founder">圈主</option>
          <option value="assistant">合伙人</option>
          <option value="vip">会员</option>
          <option value="guest">普通成员</option>
        </select>

        <button class="btn-primary" @click="openInviteModal">
          <span>邀请成员</span>
        </button>
      </div>
    </div>

    <!-- Members Table -->
    <div class="member-table-card">
      <table class="member-table">
        <thead>
          <tr>
            <th>成员基本信息</th>
            <th>手机号</th>
            <th>角色</th>
            <th>会员开通状态</th>
            <th>会员起止时间</th>
            <th>累计消费</th>
            <th>注册时间</th>
            <th>操作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="m in pagedMembers" :key="m.id">
            <td>
              <div class="member-profile-cell">
                <img :src="m.avatar" class="m-avatar" alt="Avatar" />
                <div class="m-info">
                  <div class="m-name-row">
                    <span class="m-name">{{ m.name }}</span>
                  </div>
                  <span class="m-id">ID: {{ m.id }}</span>
                </div>
              </div>
            </td>

            <td class="num-tabular">{{ m.phone }}</td>

            <!-- Role Selector directly in table -->
            <td>
              <select 
                class="role-select" 
                :value="m.role"
                :disabled="m.role === 'founder'"
                @change="(e: any) => actions.updateMemberRole(m.id, e.target.value)"
              >
                <option value="founder" disabled>圈主</option>
                <option value="assistant">合伙人</option>
                <option value="vip">会员</option>
                <option value="guest">普通成员</option>
              </select>
            </td>

            <!-- 会员开通状态 -->
            <td>
              <span
                v-if="membershipOf(m).state !== 'staff'"
                :class="['badge', membershipOf(m).state === 'active' ? 'badge-stock-up' : membershipOf(m).state === 'expired' ? 'badge-stock-down' : 'badge-muted']"
              >
                {{ membershipOf(m).label }}
              </span>
              <span v-else class="m-date">—</span>
            </td>

            <!-- 会员起止时间 -->
            <td class="num-tabular m-date">{{ membershipOf(m).range }}</td>

            <td class="num-tabular font-bold" style="color: var(--brand-green-hover);">
              ¥{{ m.spendTotal }}
            </td>

            <td class="num-tabular m-date">{{ m.joinedAt }}</td>

            <td>
              <div class="action-btn-group">
                <button class="tbl-action-btn" @click="openEditMember(m)">编辑</button>
                <button 
                  v-if="m.role !== 'founder'"
                  :class="['tbl-action-btn', { 'warn': m.status === 'active' }]"
                  @click="actions.toggleMuteMember(m.id)"
                >
                  {{ m.status === 'active' ? '禁言' : '解除禁言' }}
                </button>
              </div>
              <span v-if="m.role === 'founder'" class="founder-lock-text">系统保护</span>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- Pagination -->
      <div class="table-pagination">
        <span class="page-total">共 {{ displayedMembers.length }} 名成员 · 第 {{ currentPage }}/{{ totalPages }} 页</span>
        <div class="page-btns">
          <button class="page-btn" :disabled="currentPage === 1" @click="currentPage--">上一页</button>
          <template v-for="(p, pi) in pageNumbers" :key="pi">
            <span v-if="p === '...'" class="page-ellipsis">…</span>
            <button v-else :class="['page-btn', { active: p === currentPage }]" @click="currentPage = p as number">{{ p }}</button>
          </template>
          <button class="page-btn" :disabled="currentPage === totalPages" @click="currentPage++">下一页</button>
        </div>
      </div>
    </div>

    <!-- Edit Member Modal -->
    <div v-if="showEditMemberModal" class="modal-backdrop" @click.self="showEditMemberModal = false">
      <div class="modal-dialog">
        <div class="modal-header">
          <h3 class="modal-title">编辑成员</h3>
          <button class="modal-close-btn" @click="showEditMemberModal = false">×</button>
        </div>

        <div class="modal-body">
          <div class="form-group">
            <label class="form-label">成员头像</label>
            <div class="avatar-upload-wrap">
              <img :src="newMemberAvatar" class="avatar-preview" alt="头像" />
              <button type="button" class="avatar-upload-btn" title="上传头像" @click="avatarInput?.click()">
                <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M23 19a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h4l2-3h6l2 3h4a2 2 0 0 1 2 2z"></path>
                  <circle cx="12" cy="13" r="4"></circle>
                </svg>
              </button>
              <input ref="avatarInput" type="file" accept="image/*" class="hidden-file-input" @change="onAvatarChange" />
            </div>
          </div>

          <div class="form-group">
            <label class="form-label">成员昵称</label>
            <input type="text" v-model="newMemberName" placeholder="例如：雪球老法师" class="form-input" />
          </div>

          <div class="form-group">
            <label class="form-label">绑定手机号</label>
            <input type="text" v-model="newMemberPhone" placeholder="13800000000" class="form-input" />
          </div>

          <div class="form-group">
            <label class="form-label">权限角色</label>
            <select v-model="newMemberRole" class="form-input" :disabled="editingMember?.role === 'founder'">
              <option value="vip">年度会员</option>
              <option value="guest">普通用户</option>
              <option value="assistant">合伙人</option>
              <option value="founder" disabled>圈主</option>
            </select>
          </div>

          <div class="form-group">
            <label class="form-label">备注说明</label>
            <input type="text" v-model="newMemberNotes" placeholder="例如：机构投资顾问 / 终身会员赠送" class="form-input" />
          </div>
        </div>

        <div class="modal-footer">
          <button class="btn-secondary" @click="showEditMemberModal = false">取消</button>
          <button class="btn-primary" @click="handleSaveMember">保存修改</button>
        </div>
      </div>
    </div>

    <!-- Invite Member Modal -->
    <div v-if="showInviteModal" class="modal-backdrop" @click.self="showInviteModal = false">
      <div class="modal-dialog">
        <div class="modal-header">
          <h3 class="modal-title">邀请成员</h3>
          <button class="modal-close-btn" @click="showInviteModal = false">×</button>
        </div>

        <div class="modal-body">
          <p class="invite-tip">圈子成员需自行通过邀请链接注册加入，暂不支持手动添加。请将以下链接分享给好友：</p>
          <div class="invite-link-row">
            <input type="text" :value="inviteLink" readonly class="form-input" />
            <button class="btn-primary" @click="copyInviteLink">复制链接</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import { appState, actions } from '@/stores/community';
import { UserRole, MemberRecord } from '@/types';

const searchQuery = ref('');
const filterRole = ref<string>('all');
const showEditMemberModal = ref(false);
const showInviteModal = ref(false);
const inviteLink = ref('');

// 编辑中的成员
const editingMemberId = ref<string | null>(null);
const editingMember = computed(() => appState.members.find(m => m.id === editingMemberId.value) || null);

const newMemberName = ref('');
const newMemberPhone = ref('');
const newMemberRole = ref<UserRole>('vip');
const newMemberNotes = ref('');
const newMemberAvatar = ref('');
const avatarInput = ref<HTMLInputElement | null>(null);

const DEFAULT_AVATAR = 'https://images.unsplash.com/photo-1535713875002-d1d0cf377fde?w=100&auto=format&fit=crop&q=80';

// 头像文件选择后转成 base64 预览
const onAvatarChange = (e: Event) => {
  const file = (e.target as HTMLInputElement).files?.[0];
  if (!file) return;
  if (!file.type.startsWith('image/')) {
    actions.showToast('请选择图片文件');
    return;
  }
  const reader = new FileReader();
  reader.onload = () => {
    newMemberAvatar.value = reader.result as string;
  };
  reader.readAsDataURL(file);
  (e.target as HTMLInputElement).value = '';
};

const vipCount = computed(() => appState.members.filter(m => m.role === 'vip').length);
const assistantCount = computed(() => appState.members.filter(m => m.role === 'assistant').length);
const totalRevenue = computed(() => appState.members.reduce((sum, m) => sum + m.spendTotal, 0));

// 日期加 N 年，用于推导会员到期时间
const addYears = (date: string, years: number) => {
  const d = new Date(date);
  if (isNaN(d.getTime())) return '';
  d.setFullYear(d.getFullYear() + years);
  return d.toISOString().slice(0, 10);
};

// 会员开通展示信息：官方角色不涉及；会员默认已开通（起止=加入时间起一年）；支持 mock 显式覆盖
const membershipOf = (m: MemberRecord) => {
  if (m.role === 'founder' || m.role === 'assistant') {
    return { state: 'staff' as const, label: '—', range: '—' };
  }
  if (m.membershipStatus === 'expired') {
    return { state: 'expired' as const, label: '已过期', range: `${m.membershipStart || m.joinedAt} ~ ${m.membershipEnd || ''}` };
  }
  if (m.role === 'vip' || m.membershipStatus === 'active') {
    return {
      state: 'active' as const,
      label: '已开通',
      range: `${m.membershipStart || m.joinedAt} ~ ${m.membershipEnd || addYears(m.joinedAt, 1)}`,
    };
  }
  return { state: 'none' as const, label: '未开通', range: '—' };
};

const displayedMembers = computed(() => {
  let list = appState.members;
  if (filterRole.value !== 'all') {
    list = list.filter(m => m.role === filterRole.value);
  }
  if (searchQuery.value.trim()) {
    const q = searchQuery.value.toLowerCase();
    list = list.filter(m => m.name.toLowerCase().includes(q) || m.phone.includes(q));
  }
  return list;
});

// 分页
const pageSize = 10;
const currentPage = ref(1);
const totalPages = computed(() => Math.max(1, Math.ceil(displayedMembers.value.length / pageSize)));
const pagedMembers = computed(() => {
  const start = (currentPage.value - 1) * pageSize;
  return displayedMembers.value.slice(start, start + pageSize);
});
const pageNumbers = computed(() => {
  const total = totalPages.value;
  const cur = currentPage.value;
  if (total <= 7) return Array.from({ length: total }, (_, i) => i + 1);
  const pages = new Set<number>([1, total, cur - 1, cur, cur + 1]);
  const sorted = [...pages].filter(p => p >= 1 && p <= total).sort((a, b) => a - b);
  const out: (number | '...')[] = [];
  let prev = 0;
  for (const p of sorted) {
    if (p - prev > 1) out.push('...');
    out.push(p);
    prev = p;
  }
  return out;
});
watch([searchQuery, filterRole], () => { currentPage.value = 1; });
watch(totalPages, (v) => { if (currentPage.value > v) currentPage.value = v; });

// 打开邀请成员弹窗，生成邀请链接
const openInviteModal = () => {
  const token = Math.random().toString(36).slice(2, 10);
  inviteLink.value = `https://circle.wealth.club/r/${token}`;
  showInviteModal.value = true;
};

// 复制邀请链接
const copyInviteLink = async () => {
  try {
    await navigator.clipboard.writeText(inviteLink.value);
    actions.showToast('邀请链接已复制');
  } catch {
    actions.showToast('复制失败，请手动复制');
  }
};

// 打开编辑成员弹窗，回填数据
const openEditMember = (m: MemberRecord) => {
  editingMemberId.value = m.id;
  newMemberName.value = m.name;
  newMemberPhone.value = m.phone;
  newMemberRole.value = m.role;
  newMemberNotes.value = m.notes || '';
  newMemberAvatar.value = m.avatar || DEFAULT_AVATAR;
  showEditMemberModal.value = true;
};

// 保存成员修改
const handleSaveMember = () => {
  if (!newMemberName.value.trim()) {
    actions.showToast('请输入成员昵称');
    return;
  }
  if (!editingMemberId.value) return;
  actions.updateMember(editingMemberId.value, {
    name: newMemberName.value.trim(),
    phone: newMemberPhone.value.trim() || (editingMember.value?.phone ?? ''),
    role: newMemberRole.value,
    notes: newMemberNotes.value.trim(),
    avatar: newMemberAvatar.value,
  });
  actions.showToast(`已保存成员「${newMemberName.value.trim()}」的信息`);
  showEditMemberModal.value = false;
};
</script>

<style scoped>
.member-manage-wrap {
  width: 100%;
}

.stats-overview-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  margin-bottom: 20px;
}

.stat-card {
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: var(--radius-card);
  padding: 16px;
  box-shadow: var(--shadow-card);
  text-align: center;
}

.stat-num {
  font-size: 22px;
  font-weight: 800;
  color: var(--primary-navy);
  display: block;
}

.stat-text {
  font-size: 11px;
  color: var(--text-muted);
  margin-top: 4px;
}

.member-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #ffffff;
  border-radius: var(--radius-card);
  padding: 8px 0;
  margin-bottom: 16px;
  box-shadow: var(--shadow-card);
}

.search-input-wrap {
  display: flex;
  align-items: center;
  gap: 8px;
  height: 32px;
  background: var(--bg-subtle);
  padding: 0 12px;
  border-radius: 8px;
  width: 260px;
  box-sizing: border-box;
}

.search-input-wrap input {
  border: none;
  background: transparent;
  width: 100%;
  font-size: 13px;
  box-shadow: none;
  padding: 0;
}

.toolbar-right {
  display: flex;
  align-items: center;
  gap: 12px;
}

.role-filter-select {
  height: 32px;
  box-sizing: border-box;
  padding: 0 12px;
  font-size: 13px;
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: 8px;
  color: var(--text-secondary);
}

.member-toolbar .btn-primary {
  height: 32px;
  box-sizing: border-box;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 0 14px;
  border-radius: 8px;
}

.member-table-card {
  background: #ffffff;
  border-radius: var(--radius-card);
  box-shadow: var(--shadow-card);
  overflow-x: auto;
}

.member-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 13px;
}

.member-table th {
  text-align: left;
  padding: 12px 16px;
  background: var(--bg-subtle);
  color: var(--text-muted);
  font-size: 11px;
  font-weight: 700;
  white-space: nowrap;
}

.member-table td {
  padding: 14px 16px;
  border-top: 1px solid var(--border-light);
  vertical-align: middle;
  white-space: nowrap;
}

/* Pagination */
.table-pagination {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  border-top: 1px solid var(--border-light);
}

.page-total {
  font-size: 12px;
  color: var(--text-muted);
}

.page-btns {
  display: flex;
  align-items: center;
  gap: 6px;
}

.page-btn {
  min-width: 28px;
  height: 28px;
  padding: 0 8px;
  font-size: 12px;
  color: var(--text-secondary);
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: 6px;
  cursor: pointer;
}

.page-btn:hover:not(:disabled) {
  border-color: #2563eb;
  color: #2563eb;
}

.page-btn.active {
  background: #2563eb;
  border-color: #2563eb;
  color: #ffffff;
  font-weight: 600;
}

.page-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.page-ellipsis {
  font-size: 12px;
  color: var(--text-muted);
  padding: 0 2px;
}

.member-profile-cell {
  display: flex;
  align-items: center;
  gap: 10px;
}

.m-avatar {
  width: 38px;
  height: 38px;
  border-radius: 50%;
  object-fit: cover;
}

.m-info {
  display: flex;
  flex-direction: column;
}

.m-name-row {
  display: flex;
  align-items: center;
  gap: 6px;
}

.m-name {
  font-size: 13px;
  font-weight: 700;
  color: var(--primary-navy);
}

.m-id {
  font-size: 10px;
  color: var(--text-muted);
}

.m-date {
  font-size: 12px;
  color: var(--text-secondary);
}

.badge-muted {
  background: #f1f5f9;
  color: #94a3b8;
}

.role-select {
  padding: 5px 8px;
  font-size: 12px;
  font-weight: 600;
  border-radius: var(--radius-sm);
  background: var(--bg-subtle);
  border: 1px solid var(--border-light);
  color: var(--primary-navy);
  cursor: pointer;
}

.action-btn-group {
  display: flex;
  gap: 6px;
}

.tbl-action-btn {
  font-size: 11px;
  padding: 4px 8px;
  border-radius: 4px;
  background: var(--bg-subtle);
  color: var(--text-primary);
  font-weight: 600;
}

.tbl-action-btn.warn {
  color: #d97706;
}

.tbl-action-btn.delete {
  color: var(--danger-red);
}

.founder-lock-text {
  font-size: 11px;
  color: var(--text-muted);
}

/* Modal */
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(15, 23, 42, 0.5);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 99;
}

.modal-dialog {
  width: 460px;
  background: #ffffff;
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  overflow: hidden;
  animation: fadeIn 0.2s ease;
}

.modal-header {
  padding: 16px 20px;
  border-bottom: 1px solid var(--border-light);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.modal-title {
  font-size: 16px;
  font-weight: 800;
  color: var(--primary-navy);
}

.modal-close-btn {
  background: transparent;
  font-size: 20px;
  color: var(--text-muted);
}

.avatar-upload-wrap {
  position: relative;
  width: fit-content;
}

.avatar-preview {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  object-fit: cover;
  border: 1px solid var(--border-light);
}

.avatar-upload-btn {
  position: absolute;
  right: 0;
  bottom: 0;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: var(--primary-navy);
  color: #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
  border: 2px solid #ffffff;
  cursor: pointer;
}

.hidden-file-input {
  display: none;
}

.invite-tip {
  font-size: 13px;
  color: var(--text-secondary);
  line-height: 1.6;
  margin: 0;
}

.invite-link-row {
  display: flex;
  gap: 10px;
}

.invite-link-row .form-input {
  flex: 1;
  min-width: 0;
}

.invite-link-row .btn-primary {
  flex-shrink: 0;
  white-space: nowrap;
}

.modal-body {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-label {
  font-size: 12px;
  font-weight: 700;
  color: var(--text-secondary);
}

.form-input {
  padding: 8px 12px;
  font-size: 13px;
}

.modal-footer {
  padding: 12px 20px;
  background: var(--bg-subtle);
  border-top: 1px solid var(--border-light);
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}
</style>
