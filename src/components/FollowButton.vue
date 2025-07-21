<template>
  <div class="follow-btn">
    <n-button
        strong
        secondary
        round
        @click="() => handleFollowAction(user)"
        :loading="followLoading"
    >
      <n-icon>
        <PlusFilled v-if="!user.is_following" />
        &nbsp;
        <CheckFilled v-else />
      </n-icon>
      {{ user.is_following ? '已关注' : '关注' }}
    </n-button>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { PlusFilled, CheckFilled } from '@vicons/material';
import { useStore } from 'vuex';
import { followUser, unfollowUser } from '@/api/user';
import { useDialog } from 'naive-ui';
const emit = defineEmits(['update:follow']);

const props = defineProps({
  user: {
    type: Object,
    required: true
  },
});

const store = useStore();
const dialog = useDialog();
const followLoading = ref(false);

const handleFollowAction = (user) => {
  const targetUser = user;

  dialog.warning({
    title: '提示',
    content: `确定${targetUser.is_following ? '取消关注' : '关注'} @${targetUser.nickname} 吗？`,
    positiveText: '确定',
    negativeText: '取消',
    onPositiveClick: async () => {
      followLoading.value = true;
      try {
        if (targetUser.is_following) {
          await unfollowUser({ user_id: user.id });
        } else {
          await followUser({ user_id: user.id });
        }
        // 直接修改 props 是不推荐的，应该通过 emit 让父组件修改
        emit('update:follow', !targetUser.is_following);
        emit('follow-change'); // 新增事件，通知父组件关注状态已变化
      } catch (error) {
        console.error('操作失败:', error);
      } finally {
        followLoading.value = false;
      }
    }
  });
};
</script>