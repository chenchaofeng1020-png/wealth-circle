<template>
  <div class="content-manage-wrap animate-fade-in">
    <!-- Top Action Bar -->
    <div class="admin-top-bar">
      <div class="admin-subtabs">
        <button 
          :class="['admin-tab', { active: currentTab === 'columns' }]"
          @click="currentTab = 'columns'"
        >
          专栏管理 ({{ appState.columns.length }})
        </button>
        <button 
          :class="['admin-tab', { active: currentTab === 'courses' }]"
          @click="currentTab = 'courses'"
        >
          课程管理 ({{ appState.courses.length }})
        </button>
        <button 
          :class="['admin-tab', { active: currentTab === 'lives' }]"
          @click="currentTab = 'lives'"
        >
          直播排期 ({{ appState.liveSessions.length }})
        </button>
      </div>

      <button class="btn-primary" @click="showCreateModal = true">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <line x1="12" y1="5" x2="12" y2="19"></line>
          <line x1="5" y1="12" x2="19" y2="12"></line>
        </svg>
        <span>新建{{ currentTab === 'columns' ? '财经专栏' : currentTab === 'courses' ? '理财课程' : '直播排期' }}</span>
      </button>
    </div>

    <!-- 1. Columns Table -->
    <div v-if="currentTab === 'columns'" class="manage-table-card">
      <table class="manage-table">
        <thead>
          <tr>
            <th>专栏信息</th>
            <th>主讲作者</th>
            <th>定价/原价</th>
            <th>章节数</th>
            <th>订阅人数</th>
            <th>状态</th>
            <th>管理操作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="col in appState.columns" :key="col.id">
            <td>
              <div class="col-meta-cell">
                <img :src="col.cover" class="col-thumb" alt="Cover" />
                <div class="col-text-meta">
                  <span class="col-cell-title">{{ col.title }}</span>
                  <span class="col-cell-sub">{{ col.subtitle }}</span>
                </div>
              </div>
            </td>
            <td><strong>{{ col.author }}</strong></td>
            <td>
              <span class="num-tabular font-bold" style="color: var(--brand-green-hover);">¥{{ col.price }}</span>
              <span class="num-tabular origin-price" style="margin-left: 4px;">¥{{ col.originPrice }}</span>
            </td>
            <td class="num-tabular">{{ col.chapterCount }} 讲</td>
            <td class="num-tabular">{{ col.subscriberCount }}</td>
            <td>
              <span :class="['badge', col.status === 'published' ? 'badge-stock-up' : 'badge-guest']">
                {{ col.status === 'published' ? '已上架销售' : '草稿未公开' }}
              </span>
            </td>
            <td>
              <div class="action-btn-group">
                <button 
                  class="tbl-action-btn"
                  @click="actions.toggleColumnStatus(col.id)"
                >
                  {{ col.status === 'published' ? '下架' : '上架' }}
                </button>
                <button 
                  class="tbl-action-btn delete"
                  @click="actions.deleteColumn(col.id)"
                >
                  删除
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- 2. Courses Table -->
    <div v-if="currentTab === 'courses'" class="manage-table-card">
      <table class="manage-table">
        <thead>
          <tr>
            <th>课程信息</th>
            <th>讲师</th>
            <th>总课时</th>
            <th>难度级别</th>
            <th>学员数</th>
            <th>状态</th>
            <th>操作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="course in appState.courses" :key="course.id">
            <td>
              <div class="col-meta-cell">
                <img :src="course.cover" class="col-thumb" alt="Cover" />
                <div class="col-text-meta">
                  <span class="col-cell-title">{{ course.title }}</span>
                  <span class="col-cell-sub">{{ course.description }}</span>
                </div>
              </div>
            </td>
            <td>{{ course.instructor }}</td>
            <td class="num-tabular">{{ course.lessonsCount }} 讲 ({{ course.totalDuration }})</td>
            <td><span class="badge badge-admin">{{ course.level }}</span></td>
            <td class="num-tabular">{{ course.studentsCount }}</td>
            <td><span class="badge badge-stock-up">已发布</span></td>
            <td>
              <div class="action-btn-group">
                <button class="tbl-action-btn" @click="actions.showToast('已进入课时编辑模式')">编辑课件</button>
                <button class="tbl-action-btn delete" @click="actions.showToast('该课程已有学员购买，暂不支持直接删除')">下架</button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- 3. Lives Table -->
    <div v-if="currentTab === 'lives'" class="manage-table-card">
      <table class="manage-table">
        <thead>
          <tr>
            <th>直播主题</th>
            <th>主理嘉宾</th>
            <th>计划开播时间</th>
            <th>在线/预约人数</th>
            <th>当前状态</th>
            <th>推流控制</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="live in appState.liveSessions" :key="live.id">
            <td><strong>{{ live.title }}</strong></td>
            <td>{{ live.host }} ({{ live.hostTitle }})</td>
            <td class="num-tabular">{{ live.scheduledTime }}</td>
            <td class="num-tabular">{{ live.viewersCount }}</td>
            <td>
              <span :class="['badge', live.status === 'live' ? 'badge-stock-up' : 'badge-vip']">
                {{ live.status === 'live' ? '直播进行中' : live.status === 'upcoming' ? '预约中' : '已归档' }}
              </span>
            </td>
            <td>
              <div class="action-btn-group">
                <button class="tbl-action-btn" @click="actions.showToast('已获取 OBS 推流码与专属机位地址')">推流地址</button>
                <button class="tbl-action-btn" @click="actions.showToast('已进入导播台控制中心')">导播台</button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Create New Content Modal -->
    <div v-if="showCreateModal" class="modal-backdrop" @click.self="showCreateModal = false">
      <div class="modal-dialog">
        <div class="modal-header">
          <h3 class="modal-title">新建{{ currentTab === 'columns' ? '财经专栏' : currentTab === 'courses' ? '体系课程' : '直播排期' }}</h3>
          <button class="modal-close-btn" @click="showCreateModal = false">×</button>
        </div>

        <div class="modal-body">
          <div class="form-group">
            <label class="form-label">标题名称</label>
            <input 
              type="text" 
              v-model="newTitle" 
              placeholder="如：《2026年终大盘资产配置与现金流战法》" 
              class="form-input" 
            />
          </div>

          <div class="form-group">
            <label class="form-label">简介/核心卖点</label>
            <textarea 
              v-model="newSubtitle" 
              placeholder="概括该学习内容解决的核心理财痛点..." 
              rows="3" 
              class="form-textarea"
            ></textarea>
          </div>

          <div class="form-row-2">
            <div class="form-group">
              <label class="form-label">定价 (元)</label>
              <input type="number" v-model="newPrice" class="form-input" />
            </div>
            <div class="form-group">
              <label class="form-label">分类标签</label>
              <input type="text" v-model="newTag" placeholder="如：资产配置, 财报透视" class="form-input" />
            </div>
          </div>
        </div>

        <div class="modal-footer">
          <button class="btn-secondary" @click="showCreateModal = false">取消</button>
          <button class="btn-primary" @click="handleCreateContent">确认发布上架</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { appState, actions } from '@/stores/community';

const currentTab = ref<'columns' | 'courses' | 'lives'>('columns');
const showCreateModal = ref(false);

const newTitle = ref('');
const newSubtitle = ref('');
const newPrice = ref(199);
const newTag = ref('实盘策略');

const handleCreateContent = () => {
  if (!newTitle.value.trim()) {
    actions.showToast('请输入标题名称');
    return;
  }
  actions.createColumn({
    title: newTitle.value.trim(),
    subtitle: newSubtitle.value.trim() || '高安全边际严肃理财深度长文',
    price: Number(newPrice.value) || 199,
  });

  showCreateModal.value = false;
  newTitle.value = '';
  newSubtitle.value = '';
};
</script>

<style scoped>
.content-manage-wrap {
  width: 100%;
}

.admin-top-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: var(--radius-card);
  padding: 8px 16px;
  margin-bottom: 20px;
  box-shadow: var(--shadow-card);
}

.admin-subtabs {
  display: flex;
  gap: 8px;
}

.admin-tab {
  background: transparent;
  color: var(--text-secondary);
  font-size: 13px;
  font-weight: 600;
  padding: 8px 16px;
  border-radius: var(--radius-md);
}

.admin-tab:hover {
  background: var(--bg-subtle);
  color: var(--primary-navy);
}

.admin-tab.active {
  background: var(--primary-navy);
  color: #ffffff;
}

.manage-table-card {
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: var(--radius-card);
  box-shadow: var(--shadow-card);
  overflow-x: auto;
}

.manage-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 13px;
}

.manage-table th {
  text-align: left;
  padding: 12px 16px;
  background: var(--bg-subtle);
  color: var(--text-muted);
  font-size: 11px;
  font-weight: 700;
}

.manage-table td {
  padding: 14px 16px;
  border-top: 1px solid var(--border-light);
  vertical-align: middle;
}

.col-meta-cell {
  display: flex;
  align-items: center;
  gap: 12px;
  max-width: 320px;
}

.col-thumb {
  width: 54px;
  height: 42px;
  border-radius: 4px;
  object-fit: cover;
  flex-shrink: 0;
}

.col-text-meta {
  overflow: hidden;
}

.col-cell-title {
  font-weight: 700;
  color: var(--primary-navy);
  display: block;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.col-cell-sub {
  font-size: 11px;
  color: var(--text-muted);
  display: block;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.origin-price {
  font-size: 11px;
  color: var(--text-muted);
  text-decoration: line-through;
}

.action-btn-group {
  display: flex;
  gap: 6px;
}

.tbl-action-btn {
  font-size: 12px;
  padding: 4px 10px;
  border-radius: 4px;
  background: var(--bg-subtle);
  color: var(--text-primary);
  font-weight: 600;
}

.tbl-action-btn:hover {
  background: #e2e8f0;
}

.tbl-action-btn.delete {
  color: var(--danger-red);
}
.tbl-action-btn.delete:hover {
  background: var(--danger-red-light);
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
  width: 500px;
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

.form-row-2 {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.form-label {
  font-size: 12px;
  font-weight: 700;
  color: var(--text-secondary);
}

.form-input, .form-textarea {
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
