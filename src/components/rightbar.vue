<template>
    <div class="rightbar-wrap" v-if="!store.state.collapsedRight">
        <div class="search-wrap">
            <n-input
                round
                clearable
                placeholder="搜一搜..."
                v-model:value="keyword"
                @keyup.enter.prevent="handleSearch"
            >
                <template #prefix>
                    <n-icon :component="Search" />
                </template>
            </n-input>
        </div>
        <n-card v-if="showFollowTopics" class="hottopic-wrap" title="关注话题" embedded :bordered="false" size="small">
            <n-spin :show="loading">
                <div class="hot-tag-item" v-for="tag in followTags" :key="tag.id">
                    <router-link
                        class="hash-link"
                        :to="{
                            name: 'home',
                            query: {
                                q: tag.tag,
                                t: 'tag',
                            },
                        }"
                    >
                        #{{ tag.tag }}
                    </router-link>

                    <div class="post-num">
                        {{ formatQuoteNum(tag.quote_num) }}
                    </div>
                </div>
            </n-spin>
        </n-card>
        <n-card class="hottopic-wrap" title="热门话题" embedded :bordered="false" size="small">
            <n-spin :show="loading">
                <div class="hot-tag-item" v-for="tag in hotTags" :key="tag.id">
                    <router-link
                        class="hash-link"
                        :to="{
                            name: 'home',
                            query: {
                                q: tag.tag,
                                t: 'tag',
                            },
                        }"
                    >
                        #{{ tag.tag }}
                    </router-link>

                    <div class="post-num">
                        {{ formatQuoteNum(tag.quote_num) }}
                    </div>
                </div>
            </n-spin>
        </n-card>
      <n-card
          v-if="store.state.userInfo.id > 0"
          class="signin-card"
          title="每日签到"
          embedded
          :bordered="false"
          size="small"
      >
        <div class="signin-content">
          <!-- 日期和星期 -->
          <div class="date-info">
            <div class="current-date">{{ currentDate }}</div>
            <div class="current-week">{{ currentWeek }}</div>
          </div>



          <!-- 签到信息 -->
          <div class="signin-info">
            <div class="signin-days">连续签到: {{ consecutiveDays }} 天</div>
            <n-button
                round
                type="primary"
                @click="handleSignIn"
                :disabled="todaySigned"
            >
              {{ todaySigned ? '今日已签到' : '立即签到' }}
            </n-button>
          </div>
        </div>
      </n-card>
      <!-- 签到成功弹窗 -->
      <n-modal v-model:show="showSignInModal" preset="card" style="width: 500px">
        <n-card :bordered="false">
          <!-- 礼包图标 -->
          <div class="gift-icon">
            <n-icon size="80" color="#FFD700">
              <svg x="1753189237385" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="1590" width="200" height="200"><path d="M831.3 509.6H196.6c-2.8 0-5-2.3-5-5V385.4c0-2.8 2.3-5 5-5h634.7c2.8 0 5 2.3 5 5v119.2c0 2.7-2.3 5-5 5z" fill="#AEEEEE" p-id="1591"></path><path d="M831.3 509.6H196.6c-2.8 0-5-2.3-5-5V385.4c0-2.8 2.3-5 5-5h634.7c2.8 0 5 2.3 5 5v119.2c0 2.7-2.3 5-5 5z" fill="#AEEEEE" p-id="1592"></path><path d="M788.9 825.7H239c-1.7 0-3-1.3-3-3V512.6c0-1.6 1.3-3 3-3h549.9c1.7 0 3 1.4 3 3v310.1c0 1.7-1.4 3-3 3z" fill="#85E1DE" p-id="1593"></path><path d="M831.3 509.6H196.6c-2.8 0-5-2.3-5-5V385.4c0-2.8 2.3-5 5-5h634.7c2.8 0 5 2.3 5 5v119.2c0 2.7-2.3 5-5 5z" fill="#AEEEEE" p-id="1594"></path><path d="M816.8 380.4c0 51.3 6.2 93.7-25.4 106.6-10 4.1-21.4 4-32.4 3.8-37.4-0.6-74.9-1.1-112.3-1.5-145.1-1.7-290.2-2.1-435.4-2.7l-6.6 23h626.5c0.4 0 2.1-0.1 3.5-1.4 1.5-1.4 1.5-3.3 1.5-3.6V383.8c-0.2-0.5-0.7-1.4-1.7-2.2-1.3-1-2.7-1.2-3.3-1.2h-14.4z" fill="#6FD0CB" p-id="1595"></path><path d="M714.1 511.3c-3 15.6-5.5 44.8-7.5 59-7.9 55.1-30.1 110.9-73.9 147.1-57.3 47.4-125.6 42.1-195.4 44.2-67.3 2-134.3-2.4-201.3-2.4-1.7 1.6 0 54.3 0.1 64.2 0 1.8 2.4 2.3 2.9 2.3 6.4 0.3 549.9 0 549.9 0 0.6-0.1 1.3-0.3 2-0.7 1-0.6 1-3 1-3.9 0-7.8-1.3-127.8 0-305.8 0.1-0.3 0.4-2.3-1.1-4-1.2-1.4-2.8-1.7-3.2-1.7-24.7-7.9-69.2-20.7-73.5 1.7z" fill="#6FD0CB" p-id="1596"></path><path d="M787.6 439H160c-2.9 0-6-0.3-8.9 0-1.8 0.2 20.7 20.3 20.6 21 0.1-0.8 0-1.8 0-2.6v-14.8-96.3c0-1.8 0.1-3.7 0-5.6v-1.4-0.4c0-0.4-20.8 20.9-21 20.9 0.6 0.1 1.2 0 1.7 0 4.1 0.1 8.2 0 12.3 0h619.5c0.9 0 1.9 0.1 2.8 0 1.1-0.1-20.7-20.7-20.7-20.9-0.1 1 0 2 0 3V458c0 11.3-0.6-15.6 14.4-18.1-5.8 0.9-12.1 6.6-14.9 11.5-3.1 5.4-4.5 13.3-2.5 19.3 4.5 14 16.9 19.7 30.8 17.5 13.5-2.2 21.7-15.6 22.4-28.3 0.1-1.1 0-2.2 0-3.3v-16.2-52.8-43.8c0-9.6-1.8-18.5-9-25.8-5.6-5.6-13.4-7.9-21.2-8.1-11.4-0.3-22.7 0-34.1 0H187.7c-11.8 0-23.7-0.2-35.5 0-2.9 0.1-5.9 0.3-8.8 1-13.1 3.4-21.1 15.1-21.7 28.2-0.1 1.1 0 2.2 0 3.3v112.8c0 10.7 2.4 20.7 11.5 27.7 10.9 8.4 27.4 6.1 40.5 6.1H787.5c13.1 0 25.6-11.5 25-25-0.5-13.5-10.9-25-24.9-25z" fill="#92646E" p-id="1597"></path><path d="M744.2 755.2H262.1c-22.2 0-44.6-0.8-66.8 0-4.6 0.2 1.7-2.8 9.5 2.5l9 9c3.8 5.6 2.5 14.7 2.5 8.3v-6.3-22.4-74.1-167.1-27.7-7.5c0-7 1.5 1.8-2.5 7.7l-9 9c-7.6 5.1-14.2 2.4-9.5 2.5H676.3c22.2 0 44.6 0.8 66.8 0 4.6-0.2-1.7 2.8-9.5-2.5l-9-9c-3.8-5.6-2.5-14.7-2.5-8.3V774.4c0 7-1.5-1.8 2.5-7.7l9-9c1.3-0.5 2.6-1.1 3.9-1.6-12.8 2.6-21.4 18.5-17.5 30.8 4.4 13.6 17 20.3 30.8 17.5 13.7-2.8 21.4-16.2 21.4-29.3v-9.7V727 608.3 498.1v-27.9c0-8.3-1.8-16.9-8.2-22.8-6-5.6-13.5-8.1-21.5-8.2-16.4-0.2-32.8 0-49.2 0H196.6c-3.2 0-6.2 0.3-9.4 1-13.6 3.2-20.8 16.3-20.8 29.3V763.9c0 13-1.2 25.9 10.7 35.2 8.2 6.3 16.1 6.2 25.4 6.2H744c13.1 0 25.6-11.5 25-25-0.4-13.7-10.8-25.1-24.8-25.1z" fill="#92646E" p-id="1598"></path><path d="M490.7 310.7c0.1-2.6 0.3-5.2 0.5-7.9 0.3-3.5-1.2 6.7-0.2 1.5 0.2-1.2 0.4-2.5 0.7-3.7 1.1-5.8 2.7-11.6 4.6-17.2 0.8-2.5 1.7-5.1 2.7-7.5l1.2-3c2.1-5.4-2.4 5.2 0 0 2.5-5.5 5.1-10.9 8-16.2 5.3-9.8 11.2-19.2 17.7-28.3 1.4-2 3-4 4.4-6-4.7 6.4 0.1-0.1 1.4-1.6 3.7-4.4 7.6-8.6 11.8-12.6 2.1-2 4.3-3.9 6.6-5.8 3.9-3.2-0.3 0.8-0.7 0.5 0.1 0.1 2.9-2.1 3.2-2.3 5-3.5 10.2-6.6 15.6-9.4 0.9-0.5 1.9-0.9 2.8-1.4 0.5-0.2 1-0.4 1.4-0.7 2.5-1.1 1.9-0.9-1.6 0.7 1.7 0.3 5.7-2.1 7.4-2.7 2.5-0.8 5-1.5 7.5-2.1 2.9-0.7 6.5-0.7 9.2-1.8-4.4 1.7-4.4 0.5-2 0.3 1.6-0.1 3.1-0.3 4.7-0.3 2.6-0.1 5.2-0.1 7.8 0 1.6 0.1 3.1 0.2 4.7 0.3 2.5 0.1 1.9 1.4-1.9-0.3 2.3 1.1 5.9 1 8.4 1.7 2.2 0.6 4.3 1.3 6.4 2.1 4.7 1.7-5.5-2.7-1-0.4l2.4 1.2c2 1 3.9 2.2 5.7 3.4 1.1 0.7 2.1 1.7 3.3 2.3-5.9-3.3-2.6-2-0.9-0.6 1.8 1.6 3.6 3.3 5.3 5.1 0.8 0.9 5.7 6.6 2.8 3.1-2.9-3.4-0.1 0 0.4 0.7 0.7 1.1 1.5 2.2 2.1 3.4 1.3 2.3 2.2 4.9 3.6 7.1-0.2-0.3-2.2-6-1-2.3 0.4 1.3 0.9 2.5 1.3 3.8 0.6 2.1 1.2 4.3 1.6 6.4 0.2 0.8 0.2 1.8 0.5 2.6-1.8-7-0.5-3.7-0.3-1.3 0.2 2.7 0.2 5.4 0 8.1-0.1 0.9-0.3 1.9-0.3 2.7 0-7 0.5-3.5 0-1.2s-1.1 4.6-1.9 6.9l-1.2 3.3c-1.1 2.9 2.9-6.3 1-2.4-1 1.9-1.9 3.9-3 5.7-0.9 1.5-1.9 3-2.9 4.5-3.5 5 3.9-4.6 0-0.1-2.5 2.9-5.3 5.6-8.2 8.1-4.6 3.9 4.8-3.4 0.1 0-1.6 1.1-3.2 2.2-4.9 3.3-3.4 2.1-6.9 4-10.5 5.7-0.8 0.4-1.7 0.8-2.5 1.2-4.1 1.9 6.1-2.4 1.9-0.8-1.8 0.7-3.5 1.4-5.3 2-4 1.4-8 2.7-12.1 3.8-8 2.2-16.2 3.9-24.5 5.2l-6 0.9c-4.8 0.7 6.6-0.8 1.9-0.3l-2.4 0.3c-4.4 0.5-8.8 1-13.2 1.4-17.8 1.6-35.6 2.2-53.4 3.3-6.1 0.4-12.3 0.9-18.4 1.3-7.3 0.5-16.7-0.1-23.6 2.2-12.4 4-21.7 17.5-17.5 30.8 4 12.4 17.4 21.8 30.8 17.5-6.5 2.1-8 0.9-4.3 0.7 1.3-0.1 2.6-0.2 3.8-0.3 3.9-0.3 7.8-0.6 11.8-0.8 5.8-0.4 11.7-0.9 17.5-1.2 17-1.1 34-1.6 51-3.1 40.4-3.5 82.3-10.4 114.5-36.9 16.8-13.9 28.2-34.6 31.1-56.1 3.6-26.9-5.4-52.7-23.2-72.8-37.8-42.8-104.5-37.1-147.8-6.2-24.3 17.3-42.8 40.7-57.7 66.4-15.2 26.1-27.4 55.7-28.3 86.4-0.4 13.1 11.8 25.6 25 25 13.5-0.9 24.2-11.2 24.6-25.3z" fill="#316CFF" p-id="1599"></path><path d="M440.7 330V780.2c0 13.1 11.5 25.6 25 25 13.5-0.6 25-11 25-25v-70.9-206.9V330c0-13.1-11.5-25.6-25-25-13.5 0.6-25 10.9-25 25z" fill="#316CFF" p-id="1600"></path><path d="M269.8 614.9v78.9c0 13.1 11.5 25.6 25 25 13.5-0.6 25-11 25-25v-78.9c0-13.1-11.5-25.6-25-25-13.5 0.7-25 11-25 25z" fill="#FFFFFF" p-id="1601"></path><path d="M294.7 560.3m-18.1 0a18.1 18.1 0 1 0 36.2 0 18.1 18.1 0 1 0-36.2 0Z" fill="#FFFFFF" p-id="1602"></path><path d="M269.8 614.9v78.9c0 13.1 11.5 25.6 25 25 13.5-0.6 25-11 25-25v-78.9c0-13.1-11.5-25.6-25-25-13.5 0.7-25 11-25 25z" fill="#FFFFFF" p-id="1603"></path><path d="M294.7 560.3m-18.1 0a18.1 18.1 0 1 0 36.2 0 18.1 18.1 0 1 0-36.2 0Z" fill="#FFFFFF" p-id="1604"></path><path d="M441.4 193.6c-22-26.9-52.7-49.8-87.2-57.1-34.2-7.2-73.2 1-97.3 27.7-26.1 28.8-29.2 70.3-10.5 103.9 17.8 32.1 52.7 48.4 87.2 56 43.4 9.5 88 9.4 132.2 11.7 13.1 0.7 25.6-12 25-25-0.6-14.1-11-24.3-25-25-17.4-0.9-34.8-1.4-52.2-2.3-9-0.4-18.1-1-27.1-1.8-4.2-0.4-8.3-0.8-12.5-1.3-1.9-0.2-3.8-0.5-5.8-0.7-3-0.4 1.8 0.3 1.9 0.2-0.3 0.3-4.1-0.6-4.8-0.7-14.3-2.2-28.6-5.2-42.2-10.1-1-0.4-4.4-2.3-5.2-2 0 0 4.7 2.1 2.3 1-0.7-0.3-1.4-0.6-2.2-1-3.3-1.5-6.6-3.3-9.7-5.1-1.6-0.9-3.1-1.9-4.6-2.9-0.6-0.4-1.3-0.9-1.9-1.3-1.7-1.2-1.8-2.6 1.4 1.1-2.2-2.6-5.3-4.6-7.6-7.1-0.9-1-1.8-2.3-2.8-3.3-0.1-0.1 3.2 4.4 1.7 2.2-0.5-0.8-1.1-1.6-1.7-2.3-1.9-2.8-3.5-5.7-5-8.7-0.3-0.6-0.5-1.4-0.9-1.9-0.1-0.2 2 5.2 1.1 2.7-0.6-1.6-1.2-3.1-1.7-4.7-1-3.1-1.3-6.5-2.3-9.6-0.1-0.3 0.6 5.9 0.4 3.2-0.1-0.9-0.2-1.9-0.3-2.8-0.1-1.4-0.2-2.8-0.2-4.2 0-1.6 0-3.3 0.2-4.9 0.1-0.7 0.1-1.4 0.2-2.1 0.2-3.1-1.1 5.5-0.3 2.5 0.7-3 1.3-6 2.3-8.9 0.3-0.9 0.8-1.8 1-2.7 0-0.1-2.4 5-1.1 2.7 0.8-1.5 1.5-3 2.4-4.5 0.9-1.5 1.9-2.9 2.8-4.3 1.4-2.2-1.7 2.1-1.7 2.1 0.6-0.3 1.5-1.8 1.9-2.2 2.2-2.5 4.7-4.8 7.2-6.9 3.3-2.9-4.2 2.9-0.5 0.5 1.4-0.9 2.7-1.8 4.2-2.7 1.2-0.7 2.4-1.4 3.7-2 0.6-0.3 1.3-0.6 1.9-0.9 4.4-2.3-4.2 1.5-1.4 0.5 2.9-1 5.8-2.1 8.7-2.9 1.4-0.4 2.8-0.7 4.1-1 0.9-0.2 1.9-0.3 2.8-0.5 3.4-0.7-1.6 0.2-2.4 0.3 6.5-0.2 12.7-0.5 19.2-0.1 2.8 0.2-3.4-0.5-3.1-0.4 1.1 0.3 2.3 0.4 3.5 0.6 1.7 0.3 3.4 0.7 5 1.1 4.2 1.1 8.2 2.5 12.2 3.9 0.7 0.2 3.3 1.4 0-0.1-3.5-1.6 0 0 0.7 0.3 1.9 0.9 3.8 1.8 5.7 2.8 3.5 1.8 7 3.9 10.3 6 1.8 1.2 3.6 2.4 5.3 3.6 0.9 0.6 5.7 4.1 2.6 1.9-3-2.2-0.1-0.1 0.6 0.5 0.8 0.7 1.6 1.4 2.5 2.1 1.6 1.4 3.2 2.8 4.8 4.3 6.1 5.6 11.6 11.7 16.9 18.1 8.3 10.2 26.8 9.3 35.4 0 9.8-11 8.8-24.7-0.1-35.5z" fill="#316CFF" p-id="1605"></path></svg>
            </n-icon>
          </div>

          <!-- 签到成功标题 -->
          <n-h2 align="center" style="margin-top: 0">
            <n-text type="success">签到成功！</n-text>
          </n-h2>

          <!-- 获得的经验 -->
          <n-h3 align="center" style="margin: 16px 0">
            今日签到获得
            <n-text strong type="warning">+{{ expReward }} 经验</n-text>
          </n-h3>

          <!-- 每日一句 -->
          <n-card embedded :bordered="false" style="background-color: #f8f8f8; margin-top: 24px">
            <div class="hitokoto">
              <a id="hitokoto_text">{{ hitokotoData.hitokoto }}</a>
              <div class="hitokoto-source">
                <template v-if="hitokotoData.from || hitokotoData.from_who">
                  ——
                  <template v-if="hitokotoData.from_who">{{ hitokotoData.from_who }}</template>
                  <template v-if="hitokotoData.from_who && hitokotoData.from">《</template>
                  <template v-if="hitokotoData.from">{{ hitokotoData.from }}</template>
                  <template v-if="hitokotoData.from_who && hitokotoData.from">》</template>
                </template>
                <template v-else>—— 来自「一言」</template>
              </div>
            </div>
          </n-card>

          <!-- 关闭按钮 -->
          <template #footer>
            <n-space justify="center">
              <n-button type="primary" @click="showSignInModal = false">好的</n-button>
            </n-space>
          </template>
        </n-card>
      </n-modal>
        <n-card class="copyright-wrap" embedded :bordered="false" size="small">
            <div class="copyright">&copy; {{ store.state.profile.copyrightTop }}</div>
            <div>
                <n-space>
                    <a
                        :href="store.state.profile.copyrightLeftLink"
                        target="_blank"
                        class="hash-link"
                    >
                        {{ store.state.profile.copyrightLeft }}
                    </a>
                    <a
                        :href="store.state.profile.copyrightRightLink"
                        target="_blank"
                        class="hash-link"
                    >
                        {{ store.state.profile.copyrightRight }}
                    </a>
                </n-space>
            </div>
        </n-card>
        <div class="site-info" v-if="store.state.userInfo.is_admin" ref="userInfoElement">
            <span class="site-info-item">{{ registerUserCount }} 注册用户，{{ onlineUserCount }} 人在线，最高在线 {{ historyMaxOnline }} 人，站点上线于 {{ formatRelativeTime(serverUpTime) }}</span>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed, watch } from 'vue';
import { useStore } from 'vuex';
import { useRouter } from 'vue-router';
import { getTags } from '@/api/post';
import { getSiteInfo } from '@/api/user';
import { Search } from '@vicons/ionicons5';
import { formatRelativeTime } from '@/utils/formatTime';
import {useMessage} from "naive-ui";

const hotTags = ref<Item.TagProps[]>([]);
const followTags = ref<Item.TagProps[]>([]);
const loading = ref(false);
const keyword = ref('');
const store = useStore();
const router = useRouter();
const registerUserCount = ref(0);
const onlineUserCount = ref(0);
const historyMaxOnline = ref(0);
const serverUpTime = ref(0);
const userInfoElement = ref<HTMLElement | null>(null);
const rightFollowTopicMaxSize = Number(
  import.meta.env.VITE_RIGHT_FOLLOW_TOPIC_MAX_SIZE,
);
const rightHotTopicMaxSize = Number(
  import.meta.env.VITE_RIGHT_HOT_TOPIC_MAX_SIZE,
);
const message = useMessage()

// 当前日期和星期
const currentDate = ref('')
const currentWeek = ref('')
const hitokotoText = ref('每日一句加载中...')
const consecutiveDays = ref(0) // 连续签到天数
const todaySigned = ref(false) // 今日是否已签到
const isSigning = ref(false)
const showSignInModal = ref(false)
const expReward = ref(10) // 签到获得的经验值

interface HitokotoData {
  id: number
  hitokoto: string
  type: string
  from?: string
  from_who?: string
  creator: string
  creator_uid: string
  uuid: string
  // 其他可能用到的字段...
}

const hitokotoData = ref<HitokotoData>({
  id: 0,
  hitokoto: '每日一句加载中...',
  type: '',
  from: '',
  from_who: '',
  creator: '',
  creator_uid: '',
  uuid: ''
})

const fetchHitokoto = async () => {
  try {
    const response = await fetch('https://v1.hitokoto.cn/?c=a&c=b&c=c')
    const data = await response.json()
    hitokotoData.value = {
      id: data.id,
      hitokoto: data.hitokoto,
      type: data.type,
      from: data.from || '',
      from_who: data.from_who || '',
      creator: data.creator,
      creator_uid: data.creator_uid,
      uuid: data.uuid
    }
  } catch (error) {
    hitokotoData.value.hitokoto = '生活就像海洋，只有意志坚强的人才能到达彼岸'
    console.error('获取每日一句失败:', error)
  }
}

// 更新日期和星期
const updateDateTime = () => {
  const now = new Date()
  const year = now.getFullYear()
  const month = now.getMonth() + 1
  const date = now.getDate()

  currentDate.value = `${year}年${month}月${date}日`

  const weekDays = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六']
  currentWeek.value = weekDays[now.getDay()]
}

// 模拟签到
const handleSignIn = async () => {
  isSigning.value = true

  // 模拟API请求
  await new Promise(resolve => setTimeout(resolve, 800))

  // 签到成功
  todaySigned.value = true
  isSigning.value = false
  showSignInModal.value = true

  // 每次签到随机获得10-30经验
  expReward.value = Math.floor(Math.random() * 20) + 10

  // 刷新每日一句
  fetchHitokoto()
}

// 初始化数据
const initData = () => {
  // 这里应该从服务器获取用户签到状态
  // 模拟数据
  consecutiveDays.value = 3
  todaySigned.value = false
}

onMounted(() => {
  updateDateTime()
  fetchHitokoto()
  initData()
})

const loadSiteInfo = () => {
  getSiteInfo()
    .then((res) => {
      registerUserCount.value = res.register_user_count;
      onlineUserCount.value = res.online_user_count;
      historyMaxOnline.value = res.history_max_online;
      serverUpTime.value = res.server_up_time;
    })
    .catch((_err) => {
      // do nothing
    });
  observer.disconnect();
};
const loadHotTags = () => {
  loading.value = true;
  getTags({
    type: 'hot_extral',
    num: rightHotTopicMaxSize,
    extral_num: rightFollowTopicMaxSize,
  })
    .then((res) => {
      hotTags.value = res.topics;
      followTags.value = res.extral_topics ?? [];
      showFollowTopics.value = true;
      loading.value = false;
    })
    .catch((_err) => {
      loading.value = false;
    });
};
const formatQuoteNum = (num: number) => {
  if (num >= 1000) {
    return (num / 1000).toFixed(1) + 'k';
  }
  return num;
};
const handleSearch = () => {
  router.push({
    name: 'home',
    query: {
      q: keyword.value,
    },
  });
};
const showFollowTopics = computed({
  get: () => {
    return store.state.userLogined && followTags.value.length !== 0;
  },
  set: (newVal) => {
    // do nothing
  },
});
watch(
  () => ({
    refreshTopicFollow: store.state.refreshTopicFollow,
    userLogined: store.state.userLogined,
  }),
  (to, from) => {
    if (to.refreshTopicFollow !== from.refreshTopicFollow || to.userLogined) {
      loadHotTags();
    }
    if (store.state.userInfo.is_admin) {
      loadSiteInfo();
    }
  },
);
const observer = new IntersectionObserver(
  (entries: IntersectionObserverEntry[]) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        loadSiteInfo();
      }
    });
  },
  {
    root: null,
    rootMargin: '0px',
    threshold: 1,
  },
);
onMounted(() => {
  // 不知道为什么 store.state.userInfo.is_admin 在这里就是不起作用f*k，所以才用这么一种蹩脚的法子来凑合
  if (userInfoElement.value) {
    observer.observe(userInfoElement.value);
  }
  loadHotTags();
});

</script>

<style lang="less" scoped>
.gift-icon {
  display: flex;
  justify-content: center;
  margin: 20px 0;
}
.rightbar-wrap::-webkit-scrollbar {
  width: 0; /* 隐藏滚动条的宽度 */
  height: 0; /* 隐藏滚动条的高度 */
}
.rightbar-wrap {
    width: 240px;
    position: fixed;
    left: calc(50% + var(--content-main) / 2 + 10px);
    max-height: calc(100vh); /* 调整高度 */
    overflow: auto;
    .search-wrap {
        margin: 12px 0;
    }

    .hot-tag-item {
        line-height: 2;
        position: relative;

        .hash-link {
            width: calc(100% - 60px);
            text-overflow: ellipsis;
            white-space: nowrap;
            overflow: hidden;
            display: block;
        }

        .post-num {
            position: absolute;
            right: 0;
            top: 0;
            width: 60px;
            text-align: right;
            line-height: 2;
            opacity: 0.5;
        }
    }

    .hottopic-wrap {
        margin-bottom: 10px;
    }

    .site-info {
        margin-top: 8px;
        padding-left: 16px;
        padding-right: 16px;
        .site-info-item {
            font-size: 10px;
            opacity: 0.75;
        }
    }

    .copyright-wrap {
        .copyright {
            font-size: 12px;
            opacity: 0.75;
        }

        .hash-link {
            font-size: 12px;
        }
    }
}
.dark {
    .hottopic-wrap {
        background-color: #18181c;
    }
    .copyright-wrap {
        background-color: #18181c;
    }
}

.signin-card {
  margin-bottom: 16px;
}

.signin-content {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.date-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.current-date {
  font-size: 16px;
  font-weight: bold;
}

.current-week {
  color: #666;
}

.hitokoto {
  padding: 12px;
  background-color: #f7f7f7;
  border-radius: 4px;
  font-style: italic;
  margin: 8px 0;
}

.hitokoto-source {
  text-align: right;
  font-size: 12px;
  color: #999;
  margin-top: 4px;
}

.signin-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 8px;
}

.signin-days {
  color: #666;
}

.hash-link {
  color: inherit;
  text-decoration: none;
}

.hash-link:hover {
  color: #18a058;
}
</style>