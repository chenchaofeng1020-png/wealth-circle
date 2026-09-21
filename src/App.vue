<template>
  <div class="app-root">
    <!-- 1. Landing Introduction View (Default on initial load) -->
    <LandingView v-if="appState.appFlowState === 'landing'" />

    <!-- 2. Paywall Membership Activation View -->
    <PaywallView v-else-if="appState.appFlowState === 'paywall'" />

    <!-- 3. Authenticated Community Platform -->
    <div v-else class="app-layout">
      <!-- Left Fixed Navigation Sidebar -->
      <Sidebar />

      <!-- Right Main Scrollable Viewport -->
      <div class="main-viewport">
        <!-- TopHeader for community feeds, learning center, and standard member management -->
        <TopHeader v-if="!isWorkbenchAdminView" />

        <main :class="['page-content-wrapper', { 'workbench-mode': isWorkbenchAdminView, 'left-align-content': appState.currentView === 'admin_members' }]">
          <div :class="['content-container', { 'feed-layout': appState.currentView === 'feed', 'workbench-layout': isWorkbenchAdminView }]">
            <!-- Center Main Column -->
            <div :class="['center-main-stage', { 'feed-stage': appState.currentView === 'feed', 'workbench-stage': isWorkbenchAdminView }]">
              <PostFeed v-if="appState.currentView === 'feed'" />
              <LearningCenter 
                v-else-if="appState.currentView === 'column'" 
                initialTab="column" 
              />
              <LearningCenter 
                v-else-if="appState.currentView === 'live'" 
                initialTab="live" 
              />
              <LearningCenter 
                v-else-if="appState.currentView === 'course'" 
                initialTab="course" 
              />
              <UserCenter v-else-if="appState.currentView === 'user'" />
              <ColumnManage v-else-if="appState.currentView === 'admin_column' || appState.currentView === 'admin_content'" />
              <LiveManage v-else-if="appState.currentView === 'admin_live'" />
              <CourseManage v-else-if="appState.currentView === 'admin_course'" />
              <MemberManage v-else-if="appState.currentView === 'admin_members'" />
            </div>

            <!-- Right Information Widgets (Only displayed on 'feed' view) -->
            <Rightbar v-if="appState.currentView === 'feed'" />
          </div>
        </main>
      </div>
    </div>

    <!-- Auth Modal (Registration & Login) -->
    <AuthModal />

    <!-- Floating Global Modals -->
    <PublishModal />
    <VipJoinModal />
    <ChapterReaderModal />
    <VideoPlayerModal />
    <LogoutModal />
    <CommunityMembersModal />
    <PostDetailModal />

    <!-- Toast Notice -->
    <transition name="toast-fade">
      <div v-if="appState.toastMessage" class="global-toast">
        <div class="toast-indicator"></div>
        <span>{{ appState.toastMessage }}</span>
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { appState } from '@/stores/community';
import LandingView from './components/landing/LandingView.vue';
import PaywallView from './components/landing/PaywallView.vue';
import AuthModal from './components/auth/AuthModal.vue';
import Sidebar from './components/layout/Sidebar.vue';
import TopHeader from './components/layout/TopHeader.vue';
import Rightbar from './components/layout/Rightbar.vue';
import PostFeed from './components/community/PostFeed.vue';
import LearningCenter from './components/learning/LearningCenter.vue';
import UserCenter from './components/user/UserCenter.vue';
import ContentManage from './components/admin/ContentManage.vue';
import MemberManage from './components/admin/MemberManage.vue';
import ColumnManage from './components/admin/ColumnManage.vue';
import LiveManage from './components/admin/LiveManage.vue';
import CourseManage from './components/admin/CourseManage.vue';
import PublishModal from './components/modals/PublishModal.vue';
import VipJoinModal from './components/modals/VipJoinModal.vue';
import ChapterReaderModal from './components/modals/ChapterReaderModal.vue';
import VideoPlayerModal from './components/modals/VideoPlayerModal.vue';
import LogoutModal from './components/modals/LogoutModal.vue';
import CommunityMembersModal from './components/modals/CommunityMembersModal.vue';
import PostDetailModal from './components/modals/PostDetailModal.vue';

const isWorkbenchAdminView = computed(() => {
  return appState.currentView === 'admin_column' || 
         appState.currentView === 'admin_content' || 
         appState.currentView === 'admin_live' || 
         appState.currentView === 'admin_course';
});
</script>

<style scoped>
.app-layout {
  display: flex;
  height: 100vh;
  overflow: hidden;
  background-color: #ffffff;
}

.main-viewport {
  flex: 1;
  display: flex;
  flex-direction: column;
  height: 100vh;
  min-width: 0;
  overflow: hidden;
  background-color: #ffffff;
}

.page-content-wrapper {
  flex: 1;
  display: flex;
  justify-content: center;
  min-height: 0;
  overflow: hidden;
  padding: 20px 32px 0 32px;
  background-color: #ffffff;
}

.page-content-wrapper.workbench-mode {
  padding: 0;
  background-color: #f8fafc;
}

.page-content-wrapper.left-align-content {
  justify-content: flex-start;
}

.page-content-wrapper.left-align-content .content-container {
  max-width: 100%;
}

.content-container {
  width: 100%;
  max-width: 1100px;
  display: flex;
  gap: 24px;
  height: 100%;
  min-height: 0;
}

.content-container.feed-layout {
  max-width: 1180px;
}

.content-container.workbench-layout {
  max-width: 100%;
  gap: 0;
}

.center-main-stage {
  flex: 1;
  min-width: 0;
  height: 100%;
  overflow-y: auto;
  overscroll-behavior-y: contain;
  scrollbar-width: thin;
  scrollbar-color: transparent transparent;
  transition: scrollbar-color 0.2s ease;
}

.center-main-stage:hover {
  scrollbar-color: #cbd5e1 transparent;
}

.center-main-stage::-webkit-scrollbar {
  width: 6px;
}

.center-main-stage::-webkit-scrollbar-track {
  background: transparent;
}

.center-main-stage::-webkit-scrollbar-thumb {
  background: transparent;
  border-radius: 4px;
  transition: background-color 0.2s ease;
}

.center-main-stage:hover::-webkit-scrollbar-thumb {
  background: #cbd5e1;
}

.center-main-stage::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}

.center-main-stage:not(.workbench-stage) {
  padding-bottom: 32px;
}

.center-main-stage.feed-stage {
  padding-right: 4px;
}

.center-main-stage.workbench-stage {
  overflow: hidden;
  padding-bottom: 0;
}

/* Global Toast */
.global-toast {
  position: fixed;
  bottom: 28px;
  left: 50%;
  transform: translateX(-50%);
  background: var(--primary-navy);
  color: #ffffff;
  padding: 10px 20px;
  border-radius: var(--radius-full);
  box-shadow: var(--shadow-lg);
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  font-weight: 600;
  z-index: 9999;
  border: 1px solid rgba(255, 255, 255, 0.15);
}

.toast-indicator {
  width: 8px;
  height: 8px;
  background: var(--brand-green);
  border-radius: 50%;
}

.toast-fade-enter-active,
.toast-fade-leave-active {
  transition: all 0.25s cubic-bezier(0.16, 1, 0.3, 1);
}

.toast-fade-enter-from,
.toast-fade-leave-to {
  opacity: 0;
  transform: translate(-50%, 14px);
}
</style>
