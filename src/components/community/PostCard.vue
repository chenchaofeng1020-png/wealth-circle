<template>
  <article class="post-card animate-fade-in">
    <!-- Author Meta Row -->
    <div class="post-author-row">
      <div class="author-left">
        <img :src="post.author.avatar" class="author-avatar" alt="Avatar" />
        <div class="author-meta">
          <div class="name-row">
            <span :class="['author-name', { 'author-name-mark': post.author.name === 'Mark' }]">
              {{ post.author.name }}
            </span>
            <span v-if="post.author.role === 'founder'" class="badge badge-founder">
              圈主
            </span>
            <span v-if="post.isVipOnly" class="badge badge-vip">
              🔒 会员独享
            </span>
          </div>
          <span class="post-time">{{ post.createdAt }}</span>
        </div>
      </div>

      <!-- Right More Options (...) Dropdown -->
      <div class="author-right-menu">
        <button class="more-btn" title="更多操作" @click.stop="menuOpen = !menuOpen">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="currentColor">
            <circle cx="5" cy="12" r="1.8"></circle>
            <circle cx="12" cy="12" r="1.8"></circle>
            <circle cx="19" cy="12" r="1.8"></circle>
          </svg>
        </button>
        <transition name="menu-fade">
          <div v-if="menuOpen" class="post-menu" @click.stop>
            <div class="menu-item" @click="handleCopyLink">复制链接</div>
            <div v-if="canEditDelete" class="menu-item" @click="handleEdit">编辑</div>
            <div v-if="canManage" class="menu-item" @click="handleCollect">收进专栏</div>
            <div v-if="canManage" class="menu-item" @click="handlePin">设为置顶</div>
            <div v-if="canManage" class="menu-item" @click="handleEssence">设为精华</div>
            <div v-if="canEditDelete" class="menu-item menu-item-danger" @click="handleDelete">删除</div>
          </div>
        </transition>
      </div>
    </div>

    <!-- Post Title Row (Tags in sky blue + Title in bold text) -->
    <div v-if="post.title || (post.tags && post.tags.length)" class="post-title-row">
      <span 
        v-for="tag in post.tags" 
        :key="tag" 
        class="title-tag"
        @click.stop="actions.showToast(`已按 #${tag} 过滤`)"
      >
        #{{ tag }}
      </span>
      <h3 v-if="post.title" class="post-title-text">
        {{ post.title }}
      </h3>
    </div>

    <!-- Post Content Container -->
    <div class="post-content-container">
      <div :class="['post-text', { 'vip-locked-blur': isLockedForUser }]">
        <p 
          v-for="(paragraph, idx) in formattedParagraphs" 
          :key="idx" 
          class="content-p"
          v-html="formatMarkdown(paragraph)"
        ></p>
      </div>

      <!-- Expand / Collapse Button -->
      <div v-if="!hideExpand" class="expand-action-row">
        <span class="expand-btn" @click="isExpanded = !isExpanded">
          {{ isExpanded ? '收起全部' : '展开全部' }}
        </span>
      </div>

      <!-- Bottom Topic Tags (Pill Badges) -->
      <div v-if="post.tags && post.tags.length" class="post-tags-pill-row">
        <span 
          v-for="tag in post.tags" 
          :key="tag" 
          class="tag-pill-badge"
          @click.stop="actions.showToast(`已按话题 #${tag} 过滤`)"
        >
          {{ tag }}
        </span>
      </div>

      <!-- VIP Paywall Overlay if locked -->
      <div v-if="isLockedForUser" class="vip-lock-overlay">
        <div class="lock-icon-wrap">🔒</div>
        <div class="lock-title">本篇为「财不外露」私享会员独享研报</div>
        <p class="lock-tip">已包含深度安全边际估值模型与底仓清单</p>
        <button class="btn-vip" @click="appState.isVipJoinModalOpen = true">
          开通会员
        </button>
      </div>
    </div>

    <!-- Attached Images / Chart Preview -->
    <div v-if="post.images && post.images.length && !isLockedForUser" class="post-images-container">
      <div 
        v-for="(img, idx) in post.images" 
        :key="idx" 
        class="chart-img-wrap"
        @click="actions.showToast('点击查看高清大图')"
      >
        <img :src="img" class="feed-chart-img" alt="研究图表" />
      </div>
    </div>

    <!-- Action Bar (Likes, Comments, Favorite, Report, Share, Detail) -->
    <div class="post-action-bar">
      <div class="action-bar-left">
        <!-- Like Button -->
        <button 
          :class="['action-btn', { active: post.isLiked }]"
          title="赞"
          @click="actions.toggleLikePost(post.id)"
        >
          <svg width="18" height="18" viewBox="0 0 24 24" :fill="post.isLiked ? '#d97706' : 'none'" :stroke="post.isLiked ? '#d97706' : 'currentColor'" stroke-width="1.8">
            <path d="M14 9V5a3 3 0 0 0-3-3l-4 9v11h11.28a2 2 0 0 0 2-1.7l1.38-9a2 2 0 0 0-2-2.3zM7 22H4a2 2 0 0 1-2-2v-7a2 2 0 0 1 2-2h3"/>
          </svg>
        </button>

        <!-- Comment Button -->
        <button 
          :class="['action-btn', { active: showComments }]"
          title="讨论"
          @click="showComments = !showComments"
        >
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
            <path d="M21 11.5a8.38 8.38 0 0 1-.9 3.8 8.5 8.5 0 0 1-7.6 4.7 8.38 8.38 0 0 1-3.8-.9L3 21l1.9-5.7a8.38 8.38 0 0 1-.9-3.8 8.5 8.5 0 0 1 4.7-7.6 8.38 8.38 0 0 1 3.8-.9h.5a8.48 8.48 0 0 1 8 8v.5z"/>
          </svg>
        </button>

        <!-- Star / Favorite Button -->
        <button 
          :class="['action-btn', { 'active-gold': isFavorited }]"
          title="收藏"
          @click="toggleFavorite"
        >
          <svg width="18" height="18" viewBox="0 0 24 24" :fill="isFavorited ? '#f59e0b' : 'none'" :stroke="isFavorited ? '#f59e0b' : 'currentColor'" stroke-width="1.8">
            <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon>
          </svg>
        </button>

        <!-- Exclamation / Report Button -->
        <button 
          class="action-btn"
          title="反馈与说明"
          @click="actions.showToast('已反馈')"
        >
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
            <circle cx="12" cy="12" r="10"></circle>
            <line x1="12" y1="8" x2="12" y2="12"></line>
            <line x1="12" y1="16" x2="12.01" y2="16"></line>
          </svg>
        </button>

        <!-- Share Button -->
        <button class="action-btn" title="分享" @click="actions.showToast('已生成分享卡片')">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
            <circle cx="18" cy="5" r="3"></circle>
            <circle cx="6" cy="12" r="3"></circle>
            <circle cx="18" cy="19" r="3"></circle>
            <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
            <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
          </svg>
        </button>
      </div>

      <!-- Right: 查看详情 > -->
      <div v-if="showDetailLink" class="action-bar-right">
        <span class="detail-link" @click="actions.openPostDetail(post.id)">
          查看详情 &gt;
        </span>
      </div>
    </div>

    <!-- Likers & Featured Comment Area (as circled in red in screenshot) -->
    <div v-if="(post.likers && post.likers.length) || post.featuredComment" class="post-feedback-box">
      <!-- Likers Line -->
      <div v-if="post.likers && post.likers.length" class="likers-row">
        <span class="likers-list">
          <template v-for="(liker, idx) in post.likers" :key="liker">
            <span class="liker-name" @click="actions.showToast(`查看用户：${liker}`)">{{ liker }}</span>
            <span v-if="idx < post.likers.length - 1" class="liker-sep">、</span>
          </template>
        </span>
        <span class="likers-count-suffix"> 等 {{ post.likersCount || post.likes }}人觉得很赞</span>
      </div>

      <!-- Featured Comment Box -->
      <div v-if="post.featuredComment" class="featured-comment-box">
        <div class="fc-text-line">
          <span :class="['fc-author', { 'author-name-mark': post.featuredComment.author.name === 'Mark' }]">
            {{ post.featuredComment.author.name }}
          </span>
          <span class="fc-colon">：</span>
          <span class="fc-body">{{ post.featuredComment.content }}</span>
        </div>

        <div class="fc-meta-row">
          <span class="fc-time">{{ post.featuredComment.createdAt }}</span>
          <div class="fc-mini-actions">
            <button class="fc-mini-btn" title="回复" @click="handleReply(post.featuredComment)">
              <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
                <path d="M21 11.5a8.38 8.38 0 0 1-.9 3.8 8.5 8.5 0 0 1-7.6 4.7 8.38 8.38 0 0 1-3.8-.9L3 21l1.9-5.7a8.38 8.38 0 0 1-.9-3.8 8.5 8.5 0 0 1 4.7-7.6 8.38 8.38 0 0 1 3.8-.9h.5a8.48 8.48 0 0 1 8 8v.5z"/>
              </svg>
            </button>
            <button 
              :class="['fc-mini-btn', { active: post.featuredComment.isLiked }]" 
              title="赞" 
              @click="handleLikeFc(post.featuredComment)"
            >
              <svg width="15" height="15" viewBox="0 0 24 24" :fill="post.featuredComment.isLiked ? 'currentColor' : 'none'" stroke="currentColor" stroke-width="1.8">
                <path d="M14 9V5a3 3 0 0 0-3-3l-4 9v11h11.28a2 2 0 0 0 2-1.7l1.38-9a2 2 0 0 0-2-2.3zM7 22H4a2 2 0 0 1-2-2v-7a2 2 0 0 1 2-2h3"/>
              </svg>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Inline Comment Drawer -->
    <div v-if="showComments" class="comment-drawer">
      <!-- Input Row -->
      <div class="comment-input-row">
        <input 
          type="text" 
          v-model="commentText" 
          :placeholder="isGuestUser ? '开通会员后即可参与主题评论与讨论...' : '撰写你的理财见解，探讨标的逻辑...'"
          @keyup.enter="submitComment"
        />
        <button 
          class="btn-primary btn-comment-send" 
          :disabled="!commentText.trim()"
          @click="submitComment"
        >
          发送
        </button>
      </div>

      <!-- Comments List -->
      <div v-if="post.comments.length" class="comments-list">
        <div v-for="c in post.comments" :key="c.id" class="comment-item">
          <img :src="c.author.avatar" class="comment-avatar" alt="Avatar" />
          <div class="comment-body">
            <div class="comment-header">
              <span class="comment-name">{{ c.author.name }}</span>
              <span class="comment-time">{{ c.createdAt }}</span>
            </div>
            <p class="comment-text">{{ c.content }}</p>
          </div>
        </div>
      </div>
      <div v-else class="empty-comments">
        暂无讨论，发表你的第一条深度评论吧
      </div>
    </div>
  </article>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from 'vue';
import { Post, UserRole } from '@/types';
import { appState, actions } from '@/stores/community';

const props = withDefaults(defineProps<{
  post: Post;
  defaultShowComments?: boolean;
  showDetailLink?: boolean;
  hideExpand?: boolean;
}>(), {
  defaultShowComments: false,
  showDetailLink: true,
  hideExpand: false,
});

const showComments = ref(props.defaultShowComments);
const isExpanded = ref(props.hideExpand);
const commentText = ref('');

const formattedParagraphs = computed(() => {
  return props.post.content.split('\n').filter(p => p.trim());
});

const formatMarkdown = (text: string) => {
  if (!text) return '';
  return text.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>');
};

const isLockedForUser = computed(() => {
  if (!props.post.isVipOnly) return false;
  if (appState.user.role === 'founder' || appState.user.role === 'admin' || appState.user.role === 'vip') {
    return false;
  }
  return true;
});

const isGuestUser = computed(() => appState.user.role === 'guest');

// ---- 更多操作下拉菜单 ----
const menuOpen = ref(false);

// 是否当前帖子的创建人本人
const isCreator = computed(() => appState.user.id === props.post.author.id);
// 编辑 / 删除：圈主 或 创建人本人 可见
const canEditDelete = computed(() => appState.user.role === 'founder' || isCreator.value);
// 收进专栏 / 设为置顶 / 设为精华：圈主 或 合伙人（assistant）可见
const canManage = computed(() => appState.user.role === 'founder' || appState.user.role === 'assistant');

// 点击页面其他区域时关闭菜单
const onDocClick = () => { menuOpen.value = false; };
onMounted(() => document.addEventListener('click', onDocClick));
onBeforeUnmount(() => document.removeEventListener('click', onDocClick));

const handleCopyLink = async () => {
  menuOpen.value = false;
  try {
    await navigator.clipboard.writeText(`${location.origin}/post/${props.post.id}`);
  } catch { /* 剪贴板不可用时忽略 */ }
  actions.showToast('已复制动态链接');
};
const handleEdit = () => {
  menuOpen.value = false;
  actions.showToast('编辑功能开发中，敬请期待');
};
const handleCollect = () => {
  menuOpen.value = false;
  actions.showToast('已收进专栏');
};
const handlePin = () => {
  menuOpen.value = false;
  actions.showToast(props.post.isPinned ? '已取消置顶' : '已设为置顶');
};
const handleEssence = () => {
  menuOpen.value = false;
  actions.showToast('已设为精华');
};
const handleDelete = () => {
  menuOpen.value = false;
  actions.deletePost(props.post.id);
};

const submitComment = () => {
  if (!commentText.value.trim()) return;
  if (isGuestUser.value) {
    actions.requireVip('开通会员后即可参与主题评论与讨论');
    return;
  }
  actions.addComment(props.post.id, commentText.value.trim());
  commentText.value = '';
};

const isFavorited = ref(false);

const toggleFavorite = () => {
  isFavorited.value = !isFavorited.value;
  actions.showToast(isFavorited.value ? '已加入我的收藏' : '已取消收藏');
};

const handleReply = (fc: any) => {
  showComments.value = true;
  commentText.value = `@${fc.author.name} `;
  actions.showToast(`已快捷回复 @${fc.author.name}`);
};

const handleLikeFc = (fc: any) => {
  fc.isLiked = !fc.isLiked;
  if (fc.isLiked) {
    fc.likes = (fc.likes || 0) + 1;
    actions.showToast('已赞同精选评论');
  } else {
    fc.likes = Math.max(0, (fc.likes || 1) - 1);
    actions.showToast('已取消赞同');
  }
};
</script>

<style scoped>
.post-card {
  background: #ffffff;
  border: 1px solid #eaedf1;
  border-radius: var(--radius-card);
  padding: 16px 20px;
  margin-bottom: 12px;
  box-shadow: none;
}

.post-author-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 12px;
}

.author-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

/* 更多操作下拉菜单 */
.author-right-menu {
  position: relative;
  z-index: 5;
}

.more-btn {
  background: none;
  border: none;
  color: #94a3b8;
  padding: 4px;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.15s, background-color 0.15s;
}

.more-btn:hover {
  color: #334155;
  background-color: #f1f5f9;
}

.post-menu {
  position: absolute;
  top: 28px;
  right: 0;
  min-width: 136px;
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  box-shadow: 0 6px 24px rgba(15, 23, 42, 0.1);
  padding: 4px;
  z-index: 30;
}

.menu-item {
  padding: 8px 12px;
  font-size: 13px;
  color: #334155;
  border-radius: 6px;
  cursor: pointer;
  white-space: nowrap;
  user-select: none;
  transition: background-color 0.12s, color 0.12s;
}

.menu-item:hover {
  background: #f1f5f9;
  color: #0f172a;
}

.menu-item-danger {
  color: #dc2626;
}

.menu-item-danger:hover {
  background: #fef2f2;
  color: #b91c1c;
}

.menu-fade-enter-active,
.menu-fade-leave-active {
  transition: all 0.15s ease;
}

.menu-fade-enter-from,
.menu-fade-leave-to {
  opacity: 0;
  transform: translateY(-4px);
}

.author-avatar {
  width: 38px;
  height: 38px;
  border-radius: 50%;
  object-fit: cover;
  border: 1px solid #e2e8f0;
}

.author-meta {
  display: flex;
  flex-direction: column;
}

.name-row {
  display: flex;
  align-items: center;
  gap: 8px;
}

.author-name {
  font-size: 14px;
  font-weight: 600;
  color: #1e293b;
}

/* Amber color for Mark matching screenshot */
.author-name-mark {
  color: #d97706;
}

.badge-founder {
  background: #fef3c7;
  color: #b45309;
  font-size: 11px;
  padding: 1px 6px;
  border-radius: 4px;
  font-weight: 600;
}

.badge-vip {
  background: #ecfdf5;
  color: #059669;
  font-size: 11px;
  padding: 1px 6px;
  border-radius: 4px;
  font-weight: 600;
}

.post-time {
  font-size: 12px;
  color: #94a3b8;
  margin-top: 2px;
}

.more-btn {
  background: transparent;
  border: none;
  color: #94a3b8;
  padding: 4px;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.15s;
}

.more-btn:hover {
  color: #475569;
}

/* Post Title & Tags matching screenshot: #市场资讯 #机构观点 Title */
.post-title-row {
  display: flex;
  align-items: baseline;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 10px;
  line-height: 1.5;
}

.title-tag {
  color: #0284c7;
  font-size: 15px;
  font-weight: 500;
  cursor: pointer;
  transition: color 0.15s;
}

.title-tag:hover {
  color: #0369a1;
  text-decoration: underline;
}

.post-title-text {
  display: inline;
  font-size: 15px;
  font-weight: 700;
  color: #1e293b;
  margin: 0;
}

.post-content-container {
  position: relative;
}

.post-text {
  font-size: 14px;
  color: #334155;
  line-height: 1.68;
}

.content-p {
  margin-bottom: 8px;
}

:deep(strong) {
  font-weight: 700;
  color: #0f172a;
}

/* Expand / Collapse link matching screenshot */
.expand-action-row {
  margin: 4px 0 10px 0;
}

.expand-btn {
  color: #2563eb;
  font-size: 13.5px;
  font-weight: 500;
  cursor: pointer;
  user-select: none;
}

.expand-btn:hover {
  text-decoration: underline;
}

/* Post Attached Images / Chart */
.post-images-container {
  margin: 10px 0 14px 0;
}

.chart-img-wrap {
  display: inline-block;
  max-width: 520px;
  width: 100%;
  border-radius: 6px;
  overflow: hidden;
  border: 1px solid #e2e8f0;
  cursor: pointer;
  background: #ffffff;
}

.feed-chart-img {
  width: 100%;
  display: block;
  object-fit: cover;
}

/* Bottom Topic Tags Pill Row */
.post-tags-pill-row {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin: 10px 0 6px 0;
}

.tag-pill-badge {
  display: inline-flex;
  align-items: center;
  font-size: 12px;
  font-weight: 500;
  color: #059669;
  background: #ecfdf5;
  border: 1px solid #a7f3d0;
  border-radius: 4px;
  padding: 2px 8px;
  cursor: pointer;
  transition: all 0.15s;
}

.tag-pill-badge:hover {
  background: #d1fae5;
  border-color: #6ee7b7;
}

/* Bottom Action Bar */
.post-action-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 10px;
  margin-top: 10px;
}

.action-bar-left {
  display: flex;
  align-items: center;
  gap: 18px;
}

.action-bar-right {
  display: flex;
  align-items: center;
}

.action-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  background: transparent;
  border: none;
  color: #94a3b8;
  padding: 4px;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.15s;
}

.action-btn:hover {
  color: #475569;
}

.action-btn.active {
  color: #d97706;
}

.action-btn.active-gold {
  color: #f59e0b;
}

.detail-link {
  font-size: 13px;
  color: #94a3b8;
  cursor: pointer;
  font-weight: 500;
  transition: color 0.15s;
}

.detail-link:hover {
  color: #475569;
  text-decoration: underline;
}

/* Likers & Featured Comment Feedback Box (Screenshot red area) */
.post-feedback-box {
  margin-top: 12px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.likers-row {
  font-size: 13.5px;
  line-height: 1.6;
  color: #475569;
  word-break: break-all;
}

.liker-name {
  color: #0284c7;
  cursor: pointer;
  font-weight: 500;
  transition: color 0.15s;
}

.liker-name:hover {
  color: #0369a1;
  text-decoration: underline;
}

.liker-sep {
  color: #64748b;
}

.likers-count-suffix {
  color: #64748b;
  margin-left: 2px;
}

.featured-comment-box {
  background: #f8fafc;
  border-radius: 6px;
  padding: 10px 14px;
  border: 1px solid #f1f5f9;
}

.fc-text-line {
  font-size: 13.5px;
  line-height: 1.65;
  color: #334155;
}

.fc-author {
  font-weight: 700;
  color: #d97706;
}

.fc-colon {
  color: #475569;
}

.fc-body {
  color: #334155;
}

.fc-meta-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 6px;
}

.fc-time {
  font-size: 12px;
  color: #94a3b8;
}

.fc-mini-actions {
  display: flex;
  align-items: center;
  gap: 12px;
}

.fc-mini-btn {
  background: transparent;
  border: none;
  color: #94a3b8;
  padding: 2px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.15s;
}

.fc-mini-btn:hover {
  color: #475569;
}

.fc-mini-btn.active {
  color: #d97706;
}

/* VIP Overlay */
.vip-locked-blur {
  filter: blur(5px);
  user-select: none;
  pointer-events: none;
  max-height: 80px;
  overflow: hidden;
}

.vip-lock-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.92);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  padding: 16px;
  text-align: center;
}

.lock-icon-wrap {
  font-size: 26px;
  margin-bottom: 4px;
}

.lock-title {
  font-size: 14px;
  font-weight: 700;
  color: #1e293b;
}

.lock-tip {
  font-size: 12px;
  color: #64748b;
  margin: 4px 0 12px 0;
}

.btn-vip {
  background: linear-gradient(135deg, #f59e0b, #d97706);
  color: #ffffff;
  font-weight: 600;
  font-size: 13px;
  padding: 7px 18px;
  border-radius: 20px;
  border: none;
  cursor: pointer;
}

/* Comments Drawer */
.comment-drawer {
  margin-top: 14px;
  padding-top: 12px;
  border-top: 1px dashed #e2e8f0;
}

.comment-input-row {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
}

.comment-input-row input {
  flex: 1;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  padding: 7px 12px;
  font-size: 13px;
  outline: none;
}

.comment-input-row input:focus {
  border-color: #0284c7;
}

.btn-comment-send {
  background: #0ea5e9;
  color: #ffffff;
  border: none;
  padding: 6px 16px;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
}

.btn-comment-send:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.comment-item {
  display: flex;
  gap: 10px;
  padding: 8px 0;
  border-bottom: 1px solid #f8fafc;
}

.comment-avatar {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  object-fit: cover;
}

.comment-body {
  flex: 1;
}

.comment-header {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  margin-bottom: 2px;
}

.comment-name {
  font-weight: 600;
  color: #1e293b;
}

.comment-time {
  color: #94a3b8;
}

.comment-text {
  font-size: 13px;
  color: #334155;
  margin: 0;
}

.empty-comments {
  font-size: 12px;
  color: #94a3b8;
  text-align: center;
  padding: 12px 0;
}
</style>
