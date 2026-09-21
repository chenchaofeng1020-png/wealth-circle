<template>
  <div class="user-center-wrap animate-fade-in">
    <!-- User Profile Header Card -->
    <div class="user-hero-card">
      <div class="hero-top-bg"></div>
      <div class="hero-content">
        <div class="hero-avatar-row">
          <img :src="appState.user.avatar" class="hero-avatar" alt="Avatar" />
          <div class="hero-main-info">
            <div class="hero-name-row">
              <h2 class="hero-user-name">{{ appState.user.name }}</h2>
              <span :class="['badge', appState.user.role === 'founder' ? 'badge-founder' : 'badge-vip']">
                {{ appState.user.roleTitle }}
              </span>
              <span class="member-id-tag">UID: {{ appState.user.id }}</span>
            </div>
            <p class="hero-bio">{{ appState.user.bio }}</p>
          </div>

          <div class="hero-actions">
            <button class="btn-secondary" @click="actions.showToast('个人资料编辑已保存')">
              编辑资料
            </button>
            <button 
              class="btn-vip"
              @click="appState.isVipJoinModalOpen = true"
            >
              {{ appState.user.role === 'vip' ? '会员延期 / 续费' : '开通私享会员' }}
            </button>
            <button 
              class="btn-hero-logout"
              @click="actions.openLogoutModal"
              title="退出当前账号"
            >
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"></path>
                <polyline points="16 17 21 12 16 7"></polyline>
                <line x1="21" y1="12" x2="9" y2="12"></line>
              </svg>
              退出登录
            </button>
          </div>
        </div>

        <!-- Metric Counter Grid -->
        <div class="hero-metrics-bar">
          <div class="metric-item">
            <span class="m-val num-tabular">{{ userPosts.length }}</span>
            <span class="m-lbl">我的发布</span>
          </div>
          <div class="metric-item">
            <span class="m-val num-tabular">3</span>
            <span class="m-lbl">学习中课程</span>
          </div>
          <div class="metric-item">
            <span class="m-val num-tabular">2</span>
            <span class="m-lbl">已订阅专栏</span>
          </div>
          <div class="metric-item">
            <span class="m-val num-tabular">{{ appState.user.points }}</span>
            <span class="m-lbl">研报积分</span>
          </div>
        </div>
      </div>
    </div>

    <!-- VIP Membership Card Box -->
    <div class="membership-vip-box">
      <div class="vip-metal-card">
        <div class="vip-metal-top">
          <div class="brand-vip-sign">
            <span class="vip-logo-char">财</span>
            <span class="vip-badge-text">财不外露 · 私享会员</span>
          </div>
          <div class="vip-validity">
            {{ appState.user.vipExpireAt ? `有效期至：${appState.user.vipExpireAt}` : '未开通' }}
          </div>
        </div>

        <div class="vip-rights-grid">
          <div class="right-pill">✓ 全站专栏免费研读</div>
          <div class="right-pill">✓ 每周实战复盘直播连麦</div>
          <div class="right-pill">✓ 核心底仓估值底清单下载</div>
          <div class="right-pill">✓ 圈主1对1理财体检指导</div>
        </div>

        <div class="vip-metal-bottom">
          <div class="vip-price-indicator">
            <span class="num-tabular vip-price-digit">¥899</span>
            <span class="vip-unit">/ 年（折合每天不到 2.5 元）</span>
          </div>
          <button class="btn-vip-metal" @click="appState.isVipJoinModalOpen = true">
            {{ appState.user.role === 'vip' ? '立即续费 享老会员专属优惠' : '立即付费加入社区' }}
          </button>
        </div>
      </div>
    </div>

    <!-- Sub-tab Switching for User Center -->
    <div class="user-subtabs-nav">
      <button 
        :class="['subtab-btn', { active: currentSubTab === 'posts' }]"
        @click="currentSubTab = 'posts'"
      >
        我的发布记录 ({{ userPosts.length }})
      </button>
      <button 
        :class="['subtab-btn', { active: currentSubTab === 'study' }]"
        @click="currentSubTab = 'study'"
      >
        我的学习记录与资产 (5)
      </button>
      <button 
        :class="['subtab-btn', { active: currentSubTab === 'orders' }]"
        @click="currentSubTab = 'orders'"
      >
        会员与付费订单
      </button>
    </div>

    <!-- 1. Posts Tab -->
    <div v-if="currentSubTab === 'posts'" class="user-tab-content">
      <div v-if="userPosts.length" class="user-posts-list">
        <PostCard 
          v-for="post in userPosts" 
          :key="post.id" 
          :post="post" 
        />
      </div>
      <div v-else class="empty-state-box">
        <p>你还没有在社区发布过主题思考</p>
        <button class="btn-primary" @click="appState.isPublishModalOpen = true">
          立即发布第一条动态
        </button>
      </div>
    </div>

    <!-- 2. Study Records Tab -->
    <div v-if="currentSubTab === 'study'" class="user-tab-content">
      <div class="study-section-title">正在研习的系统课程</div>
      <div class="study-course-list">
        <div v-for="c in appState.courses" :key="c.id" class="study-course-item">
          <img :src="c.cover" class="study-cover" alt="Course" />
          <div class="study-info">
            <div class="study-title">{{ c.title }}</div>
            <div class="study-progress-txt">已完成 {{ c.completedLessons }} / {{ c.lessonsCount }} 讲</div>
            <div class="study-bar-track">
              <div class="study-bar-fill" :style="{ width: `${(c.completedLessons / c.lessonsCount) * 100}%` }"></div>
            </div>
          </div>
          <button class="btn-primary btn-study-continue" @click="actions.playLesson(c.title, c.lessons[0].title)">
            继续学习
          </button>
        </div>
      </div>

      <div class="study-section-title" style="margin-top: 24px;">已订阅专栏长文</div>
      <div class="study-column-list">
        <div v-for="col in appState.columns" :key="col.id" class="study-col-item">
          <div class="study-col-title">{{ col.title }}</div>
          <div class="study-col-meta">作者：{{ col.author }} · 共 {{ col.chapterCount }} 讲</div>
          <button class="btn-secondary" style="margin-left: auto;" @click="actions.openChapter(col.title, col.chapters[0].title, col.chapters[0].summary)">
            阅读专栏
          </button>
        </div>
      </div>
    </div>

    <!-- 3. Orders Tab -->
    <div v-if="currentSubTab === 'orders'" class="user-tab-content">
      <div class="orders-table-card">
        <table class="orders-table">
          <thead>
            <tr>
              <th>订单编号</th>
              <th>购买服务/权益</th>
              <th>支付金额</th>
              <th>支付时间</th>
              <th>状态</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td class="num-tabular">#ORD-20240918-01</td>
              <td><strong>「财不外露」年度白金私享VIP会员</strong></td>
              <td class="num-tabular font-bold" style="color: var(--brand-green-hover);">¥899.00</td>
              <td class="num-tabular">2024-09-18 14:20</td>
              <td><span class="badge badge-stock-up">支付成功</span></td>
            </tr>
            <tr>
              <td class="num-tabular">#ORD-20240315-88</td>
              <td>专栏《严肃理财：从零搭建全天候家庭资产负债表》</td>
              <td class="num-tabular font-bold" style="color: var(--brand-green-hover);">¥299.00</td>
              <td class="num-tabular">2024-03-15 10:12</td>
              <td><span class="badge badge-stock-up">已完成</span></td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Account Settings & Logout Card -->
    <div class="account-safety-card">
      <div class="safety-card-left">
        <div class="safety-title">账号与安全设置</div>
        <div class="safety-desc">当前登录：<strong>{{ appState.user.name }}</strong> (UID: {{ appState.user.id }}) ｜ 账号状态：已激活 ｜ 会员权限：{{ appState.user.roleTitle }}</div>
      </div>
      <button class="btn-safety-logout" @click="actions.openLogoutModal">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"></path>
          <polyline points="16 17 21 12 16 7"></polyline>
          <line x1="21" y1="12" x2="9" y2="12"></line>
        </svg>
        退出当前账号
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { appState, actions } from '@/stores/community';
import PostCard from '../community/PostCard.vue';

const currentSubTab = ref<'posts' | 'study' | 'orders'>('posts');

const userPosts = computed(() => {
  return appState.posts.filter(p => p.author.id === appState.user.id || p.author.name === appState.user.name);
});
</script>

<style scoped>
.user-center-wrap {
  width: 100%;
}

.user-hero-card {
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: var(--radius-card);
  overflow: hidden;
  box-shadow: none;
  margin-bottom: 20px;
}

.hero-top-bg {
  height: 90px;
  background: linear-gradient(135deg, #0f172a 0%, #1e293b 50%, #334155 100%);
}

.hero-content {
  padding: 0 24px 20px 24px;
  position: relative;
  margin-top: -36px;
}

.hero-avatar-row {
  display: flex;
  align-items: flex-end;
  gap: 18px;
  margin-bottom: 16px;
}

.hero-avatar {
  width: 76px;
  height: 76px;
  border-radius: 50%;
  border: 4px solid #ffffff;
  object-fit: cover;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
}

.hero-main-info {
  flex: 1;
}

.hero-name-row {
  display: flex;
  align-items: center;
  gap: 8px;
}

.hero-user-name {
  font-size: 20px;
  font-weight: 800;
  color: var(--primary-navy);
}

.member-id-tag {
  font-size: 11px;
  color: var(--text-muted);
  background: var(--bg-subtle);
  padding: 2px 6px;
  border-radius: 4px;
}

.hero-bio {
  font-size: 13px;
  color: var(--text-secondary);
  margin-top: 4px;
}

.hero-actions {
  display: flex;
  gap: 8px;
  align-items: center;
}

.btn-hero-logout {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 14px;
  border-radius: 8px;
  background: #ffffff;
  border: 1px solid #cbd5e1;
  color: #64748b;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: all var(--transition-fast);
}

.btn-hero-logout:hover {
  background: #fef2f2;
  border-color: #fca5a5;
  color: #ef4444;
}

.hero-metrics-bar {
  display: flex;
  background: var(--bg-subtle);
  border-radius: var(--radius-md);
  padding: 12px 20px;
  margin-top: 14px;
}

.metric-item {
  flex: 1;
  text-align: center;
}

.metric-item:not(:last-child) {
  border-right: 1px solid var(--border-light);
}

.m-val {
  font-size: 18px;
  font-weight: 800;
  color: var(--primary-navy);
  display: block;
}

.m-lbl {
  font-size: 11px;
  color: var(--text-muted);
  margin-top: 2px;
}

/* Membership Card */
.membership-vip-box {
  margin-bottom: 20px;
}

.vip-metal-card {
  background: linear-gradient(135deg, #1e293b 0%, #0f172a 100%);
  border: 1px solid rgba(245, 158, 11, 0.4);
  border-radius: var(--radius-card);
  padding: 22px 26px;
  color: #ffffff;
  box-shadow: none;
  position: relative;
  overflow: hidden;
}

.vip-metal-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}

.brand-vip-sign {
  display: flex;
  align-items: center;
  gap: 10px;
}

.vip-logo-char {
  width: 28px;
  height: 28px;
  background: var(--vip-gold-gradient);
  border-radius: 6px;
  color: #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 900;
  font-size: 15px;
}

.vip-badge-text {
  font-size: 16px;
  font-weight: 800;
  color: #fcd34d;
  letter-spacing: 0.5px;
}

.vip-validity {
  font-size: 12px;
  color: #94a3b8;
}

.vip-rights-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
  margin-bottom: 18px;
}

.right-pill {
  font-size: 12px;
  color: #e2e8f0;
  background: rgba(255, 255, 255, 0.06);
  padding: 6px 12px;
  border-radius: 6px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.vip-metal-bottom {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 14px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.vip-price-digit {
  font-size: 26px;
  font-weight: 900;
  color: #fcd34d;
}

.vip-unit {
  font-size: 12px;
  color: #94a3b8;
  margin-left: 6px;
}

.btn-vip-metal {
  background: var(--vip-gold-gradient);
  color: #ffffff;
  font-weight: 800;
  font-size: 13px;
  padding: 10px 20px;
  border-radius: var(--radius-md);
  box-shadow: 0 4px 14px rgba(217, 119, 6, 0.35);
}
.btn-vip-metal:hover {
  filter: brightness(1.1);
  transform: translateY(-1px);
}

/* Sub tabs */
.user-subtabs-nav {
  display: flex;
  gap: 8px;
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: var(--radius-card);
  padding: 6px 12px;
  margin-bottom: 16px;
  box-shadow: var(--shadow-card);
}

.subtab-btn {
  background: transparent;
  color: var(--text-secondary);
  font-size: 13px;
  font-weight: 600;
  padding: 8px 16px;
  border-radius: var(--radius-md);
}

.subtab-btn.active {
  background: var(--primary-navy);
  color: #ffffff;
}

.study-section-title {
  font-size: 14px;
  font-weight: 800;
  color: var(--primary-navy);
  margin-bottom: 12px;
}

.study-course-list, .study-column-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.study-course-item, .study-col-item {
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: var(--radius-md);
  padding: 14px 18px;
  display: flex;
  align-items: center;
  gap: 16px;
  box-shadow: var(--shadow-card);
}

.study-cover {
  width: 70px;
  height: 50px;
  border-radius: 6px;
  object-fit: cover;
}

.study-info {
  flex: 1;
}

.study-title {
  font-size: 14px;
  font-weight: 700;
  color: var(--primary-navy);
}

.study-progress-txt {
  font-size: 11px;
  color: var(--text-muted);
  margin-top: 2px;
}

.study-bar-track {
  height: 5px;
  background: var(--bg-subtle);
  border-radius: 99px;
  overflow: hidden;
  margin-top: 6px;
  max-width: 240px;
}

.study-bar-fill {
  height: 100%;
  background: var(--brand-green);
  border-radius: 99px;
}

.btn-study-continue {
  padding: 7px 14px;
  font-size: 12px;
}

/* Orders Table */
.orders-table-card {
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: var(--radius-card);
  padding: 16px;
  box-shadow: none;
  overflow-x: auto;
}

.orders-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 13px;
}

.orders-table th {
  text-align: left;
  padding: 10px 14px;
  background: var(--bg-subtle);
  color: var(--text-muted);
  font-size: 11px;
  font-weight: 700;
}

.orders-table td {
  padding: 12px 14px;
  border-top: 1px solid var(--border-light);
  color: var(--text-primary);
}

.empty-state-box {
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: var(--radius-md);
  padding: 40px;
  text-align: center;
  color: var(--text-muted);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

/* Account Safety Card */
.account-safety-card {
  margin-top: 24px;
  background: #ffffff;
  border: 1px solid var(--border-light);
  border-radius: var(--radius-card);
  padding: 20px 24px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  box-shadow: none;
}

.safety-card-left {
  flex: 1;
}

.safety-title {
  font-size: 15px;
  font-weight: 700;
  color: var(--primary-navy);
  margin-bottom: 4px;
}

.safety-desc {
  font-size: 13px;
  color: var(--text-muted);
}

.safety-desc strong {
  color: var(--text-primary);
}

.btn-safety-logout {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 9px 18px;
  border-radius: 8px;
  background: #fef2f2;
  border: 1px solid #fee2e2;
  color: #ef4444;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
  transition: all var(--transition-fast);
  flex-shrink: 0;
}

.btn-safety-logout:hover {
  background: #fee2e2;
  border-color: #fca5a5;
  color: #dc2626;
}
</style>
