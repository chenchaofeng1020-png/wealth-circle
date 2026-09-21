<template>
  <div class="editor-box-card">
    <!-- Top Input Box (Light gray box matching screenshot) -->
    <div 
      class="editor-input-box" 
      :class="{ expanded: isExpanded }"
      @click="expandEditor"
    >
      <img :src="appState.user.avatar" class="editor-avatar" alt="Avatar" />
      
      <!-- Collapsed placeholder -->
      <div v-if="!isExpanded" class="placeholder-text">
        点击发表主题...
      </div>

      <!-- Expanded editing area -->
      <div v-else class="expanded-form" @click.stop>
        <textarea 
          v-model="content" 
          placeholder="分享你的宏观洞察、研报思考或投资见解... 支持 #标签 和 Markdown **加粗**" 
          rows="3" 
          class="content-textarea"
          ref="textareaRef"
        ></textarea>

        <!-- Selected Tags Preview -->
        <div v-if="selectedTags.length || attachedImage" class="attachments-row">
          <span 
            v-for="tag in selectedTags" 
            :key="tag" 
            class="tag-chip"
            @click.stop="removeTag(tag)"
          >
            #{{ tag }} ×
          </span>
          <span v-if="attachedImage" class="img-chip" @click.stop="attachedImage = ''">
            🖼️ 已附图片 ×
          </span>
        </div>
      </div>
    </div>

    <!-- Bottom Toolbar -->
    <div class="editor-toolbar">
      <div class="toolbar-left">
        <!-- Emoji Button -->
        <button class="tool-icon-btn" title="插入表情" @click="handleToolClick('emoji')">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="12" cy="12" r="10"></circle>
            <path d="M8 14s1.5 2 4 2 4-2 4-2"></path>
            <line x1="9" y1="9" x2="9.01" y2="9"></line>
            <line x1="15" y1="9" x2="15.01" y2="9"></line>
          </svg>
        </button>

        <!-- Picture Button -->
        <button class="tool-icon-btn" title="插入图片" @click="handleToolClick('image')">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
            <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
            <circle cx="8.5" cy="8.5" r="1.5"></circle>
            <polyline points="21 15 16 10 5 21"></polyline>
          </svg>
        </button>

        <!-- Document Button -->
        <button class="tool-icon-btn" title="添加文档附件" @click="handleToolClick('doc')">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
            <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
            <polyline points="14 2 14 8 20 8"></polyline>
            <line x1="16" y1="13" x2="8" y2="13"></line>
            <line x1="16" y1="17" x2="8" y2="17"></line>
            <polyline points="10 9 9 9 8 9"></polyline>
          </svg>
        </button>

        <!-- Bold Button -->
        <button class="tool-icon-btn bold-btn" title="加粗文本" @click="handleToolClick('bold')">
          <span class="bold-text-icon">B</span>
        </button>

        <!-- Hash Tag Button -->
        <button class="tool-icon-btn hash-btn" title="添加标签" @click="handleToolClick('hash')">
          <svg width="19" height="19" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="4" y1="9" x2="20" y2="9"></line>
            <line x1="4" y1="15" x2="20" y2="15"></line>
            <line x1="10" y1="3" x2="8" y2="21"></line>
            <line x1="16" y1="3" x2="14" y2="21"></line>
          </svg>
        </button>
      </div>

      <!-- Right Action Controls (Visible when expanded) -->
      <div v-if="isExpanded" class="toolbar-right">
        <button 
          :class="['btn-vip-toggle', { active: isVipOnly }]" 
          :title="isVipOnly ? '本主题仅VIP会员可查看正文' : '点击设为仅VIP会员可见'"
          @click="isVipOnly = !isVipOnly"
        >
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect>
            <path d="M7 11V7a5 5 0 0 1 10 0v4"></path>
          </svg>
          <span>仅会员可见</span>
        </button>
        <button class="btn-cancel" @click="cancelEdit">
          取消
        </button>
        <button 
          class="btn-publish" 
          :disabled="!content.trim()"
          @click="handlePublish"
        >
          发布
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick } from 'vue';
import { appState, actions } from '@/stores/community';
import feedChart from '@/assets/feed_chart.png';

const isExpanded = ref(false);
const content = ref('');
const selectedTags = ref<string[]>([]);
const attachedImage = ref('');
const isVipOnly = ref(false);
const textareaRef = ref<HTMLTextAreaElement | null>(null);

const expandEditor = () => {
  if (appState.user.role === 'guest') {
    actions.requireVip('开通会员后即可发布主题，与圈友深度交流');
    return;
  }
  if (!isExpanded.value) {
    isExpanded.value = true;
    nextTick(() => {
      textareaRef.value?.focus();
    });
  }
};

const handleToolClick = (tool: 'emoji' | 'image' | 'doc' | 'bold' | 'hash') => {
  isExpanded.value = true;
  nextTick(() => {
    if (tool === 'emoji') {
      content.value += ' 😊 ';
      textareaRef.value?.focus();
    } else if (tool === 'image') {
      attachedImage.value = feedChart;
      actions.showToast('已添加示例图表附件');
      textareaRef.value?.focus();
    } else if (tool === 'doc') {
      actions.showToast('支持上传 PDF / Word 等研究文件');
    } else if (tool === 'bold') {
      content.value += ' **重点内容** ';
      textareaRef.value?.focus();
    } else if (tool === 'hash') {
      const defaultTag = '市场资讯';
      if (!selectedTags.value.includes(defaultTag)) {
        selectedTags.value.push(defaultTag);
      }
      textareaRef.value?.focus();
    }
  });
};

const removeTag = (tag: string) => {
  selectedTags.value = selectedTags.value.filter(t => t !== tag);
};

const cancelEdit = () => {
  isExpanded.value = false;
  content.value = '';
  selectedTags.value = [];
  attachedImage.value = '';
  isVipOnly.value = false;
};

const handlePublish = () => {
  const text = content.value.trim();
  if (!text) return;
  if (appState.user.role === 'guest') {
    actions.requireVip('开通会员后即可发布主题，与圈友深度交流');
    return;
  }

  const vipOnly = isVipOnly.value;
  actions.publishPost({
    title: '',
    content: content.value.trim(),
    tags: selectedTags.value.length ? [...selectedTags.value] : ['市场资讯'],
    stocks: [],
    isVipOnly: vipOnly,
  });

  cancelEdit();
  actions.showToast(vipOnly ? '主题发表成功！本主题已设为仅会员可见' : '主题发表成功！');
};
</script>

<style scoped>
.editor-box-card {
  background: #ffffff;
  border: 1px solid #eaedf1;
  border-radius: var(--radius-card);
  padding: 12px 16px;
  margin-bottom: 12px;
  box-shadow: none;
}

/* Gray input container matching screenshot */
.editor-input-box {
  background: #f4f5f7;
  border-radius: 6px;
  padding: 10px 14px;
  display: flex;
  align-items: center;
  gap: 12px;
  cursor: pointer;
  min-height: 52px;
  transition: background 0.15s ease, min-height 0.2s ease;
}

.editor-input-box:hover {
  background: #eff1f5;
}

.editor-input-box.expanded {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  cursor: default;
  align-items: flex-start;
  min-height: 128px;
}

.editor-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  object-fit: cover;
  flex-shrink: 0;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.08);
}

.placeholder-text {
  font-size: 14px;
  color: #8c939d;
  user-select: none;
}

/* Expanded editing form */
.expanded-form {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
}

.content-textarea {
  width: 100%;
  border: none;
  background: transparent;
  padding: 4px 0;
  font-size: 14px;
  color: #1e293b;
  line-height: 1.6;
  resize: vertical;
  min-height: 70px;
  outline: none;
}

.attachments-row {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: 4px;
}

.tag-chip {
  font-size: 12px;
  background: #e0f2fe;
  color: #0284c7;
  padding: 2px 8px;
  border-radius: 12px;
  cursor: pointer;
  font-weight: 500;
}

.img-chip {
  font-size: 12px;
  background: #f1f5f9;
  color: #475569;
  padding: 2px 8px;
  border-radius: 12px;
  cursor: pointer;
}

/* Bottom Toolbar */
.editor-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 10px;
  padding: 2px 4px 0 4px;
}

.toolbar-left {
  display: flex;
  align-items: center;
  gap: 14px;
}

.tool-icon-btn {
  background: none;
  border: none;
  color: #64748b;
  padding: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  cursor: pointer;
  transition: color 0.15s, background-color 0.15s;
}

.tool-icon-btn:hover {
  color: #1e293b;
  background-color: #f1f5f9;
}

.bold-text-icon {
  font-family: Georgia, serif;
  font-weight: 800;
  font-size: 17px;
  line-height: 1;
  color: #64748b;
}

.tool-icon-btn:hover .bold-text-icon {
  color: #1e293b;
}

.toolbar-right {
  display: flex;
  align-items: center;
  gap: 8px;
}

.btn-cancel {
  background: none;
  border: 1px solid #cbd5e1;
  color: #64748b;
  font-size: 13px;
  padding: 5px 14px;
  border-radius: 4px;
  cursor: pointer;
}

.btn-vip-toggle {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  background: none;
  border: 1px solid #fcd34d;
  color: #b45309;
  font-size: 12px;
  font-weight: 600;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.15s;
}

.btn-vip-toggle.active {
  background: linear-gradient(135deg, #fef3c7 0%, #fde68a 100%);
  color: #78350f;
  font-weight: 700;
}

.btn-vip-toggle:hover {
  background: #fef3c7;
}

.btn-cancel:hover {
  background: #f8fafc;
  color: #334155;
}

.btn-publish {
  background: #0ea5e9;
  border: none;
  color: #ffffff;
  font-size: 13px;
  font-weight: 600;
  padding: 5px 16px;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.15s;
}

.btn-publish:hover:not(:disabled) {
  background: #0284c7;
}

.btn-publish:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
