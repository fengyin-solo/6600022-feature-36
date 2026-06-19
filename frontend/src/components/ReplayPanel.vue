<template>
  <div class="bg-gray-900 rounded-xl p-4 border border-gray-700">
    <div class="flex justify-between items-center mb-3">
      <h3 class="text-lg font-bold text-green-400">棋谱回放</h3>
      <span class="text-xs text-gray-500">{{ store.recordCount }} / {{ store.maxRecords }} · 收藏 {{ store.favoriteCount }}</span>
    </div>

    <div v-if="store.status !== 'replaying'">
      <div v-if="store.gameRecords.length > 0" class="flex gap-1 mb-3">
        <button
          v-for="tab in tabs"
          :key="tab.value"
          @click="activeTab = tab.value"
          class="flex-1 py-1.5 text-xs rounded transition-colors"
          :class="activeTab === tab.value ? 'bg-green-600 text-white' : 'bg-gray-800 text-gray-400 hover:bg-gray-700'"
        >
          {{ tab.label }}
        </button>
      </div>

      <p v-if="filteredRecords.length === 0" class="text-gray-500 text-sm">
        {{ activeTab === 'favorite' ? '暂无收藏棋谱' : '暂无棋谱记录' }}
      </p>
      <div v-else class="space-y-2 max-h-64 overflow-y-auto">
        <div
          v-for="record in filteredRecords"
          :key="record.id"
          class="bg-gray-800 rounded-lg p-3 border border-gray-700"
          :class="record.isFavorite ? 'border-yellow-600/50' : ''"
        >
          <div class="flex items-start justify-between gap-2">
            <div class="flex-1 min-w-0 cursor-pointer" @click="store.startReplay(record)">
              <div class="flex items-center gap-2 flex-wrap">
                <span class="text-sm text-gray-300">{{ record.createdAt }}</span>
                <span
                  class="text-xs px-2 py-0.5 rounded-full"
                  :class="record.winner === 1 ? 'bg-gray-700 text-gray-200' : record.winner === 2 ? 'bg-white text-black' : 'bg-yellow-600 text-white'"
                >
                  {{ record.winner === 1 ? '黑棋胜' : record.winner === 2 ? '白棋胜' : '平局' }}
                </span>
                <span v-if="record.isFavorite" class="text-yellow-400 text-xs" title="已收藏">★</span>
                <span v-if="isFrequentlyUsed(record)" class="text-xs px-1.5 py-0.5 rounded bg-green-900/50 text-green-400 border border-green-700/50">常用</span>
              </div>
              <div class="text-xs text-gray-500 mt-1">
                共 {{ record.moves.length }} 手 · 查看 {{ record.viewCount }} 次
              </div>
            </div>
            <div class="flex items-center gap-1 flex-shrink-0">
              <button
                @click.stop="store.toggleFavorite(record.id)"
                class="p-1.5 rounded transition-colors"
                :class="record.isFavorite ? 'text-yellow-400 hover:bg-yellow-900/30' : 'text-gray-500 hover:text-yellow-400 hover:bg-gray-700'"
                :title="record.isFavorite ? '取消收藏' : '收藏'"
              >
                <svg class="w-4 h-4" :fill="record.isFavorite ? 'currentColor' : 'none'" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11.049 2.927c.3-.921 1.603-.921 1.902 0l1.519 4.674a1 1 0 00.95.69h4.915c.969 0 1.371 1.24.588 1.81l-3.976 2.888a1 1 0 00-.363 1.118l1.518 4.674c.3.922-.755 1.688-1.538 1.118l-3.976-2.888a1 1 0 00-1.176 0l-3.976 2.888c-.783.57-1.838-.197-1.538-1.118l1.518-4.674a1 1 0 00-.363-1.118l-3.976-2.888c-.784-.57-.38-1.81.588-1.81h4.914a1 1 0 00.951-.69l1.519-4.674z"/>
                </svg>
              </button>
              <button
                @click.stop="handleDelete(record)"
                class="p-1.5 rounded text-gray-500 hover:text-red-400 hover:bg-red-900/20 transition-colors"
                title="删除"
              >
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-else>
      <div class="text-center mb-3">
        <span class="text-gray-400 text-sm">第 {{ store.replayIndex }} / {{ store.replayMoves.length }} 手</span>
      </div>

      <div class="w-full bg-gray-800 rounded-full h-2 mb-4">
        <div
          class="bg-green-500 h-2 rounded-full transition-all"
          :style="{ width: `${(store.replayIndex / store.replayMoves.length) * 100}%` }"
        />
      </div>

      <div class="flex items-center justify-center gap-2 mb-4">
        <button @click="store.replayGoToStart()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="回到开始">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 19l-7-7 7-7m8 14l-7-7 7-7"/></svg>
        </button>
        <button @click="store.replayStepBack()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="上一步">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"/></svg>
        </button>
        <button @click="store.toggleReplayPlay()" class="p-3 bg-green-600 rounded-lg hover:bg-green-500 text-white transition-colors" :title="store.isReplayPlaying ? '暂停' : '播放'">
          <svg v-if="!store.isReplayPlaying" class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M8 5v14l11-7z"/></svg>
          <svg v-else class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M6 4h4v16H6zM14 4h4v16h-4z"/></svg>
        </button>
        <button @click="store.replayStepForward()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="下一步">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/></svg>
        </button>
        <button @click="store.replayGoToEnd()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="跳到结尾">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 5l7 7-7 7M5 5l7 7-7 7"/></svg>
        </button>
      </div>

      <div class="flex items-center justify-center gap-2 mb-4">
        <span class="text-xs text-gray-500">速度:</span>
        <button
          v-for="speed in speeds"
          :key="speed.value"
          @click="store.setReplaySpeed(speed.value)"
          class="px-2 py-1 text-xs rounded transition-colors"
          :class="store.replaySpeed === speed.value ? 'bg-green-600 text-white' : 'bg-gray-800 text-gray-400 hover:bg-gray-700'"
        >
          {{ speed.label }}
        </button>
      </div>

      <button
        @click="store.stopReplay()"
        class="w-full py-2 bg-red-600/20 border border-red-600/50 text-red-400 rounded-lg hover:bg-red-600/30 transition-colors text-sm"
      >
        退出回放
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import { useGameStore } from '../store/game';
import type { GameRecord } from '../types';

const FREQUENT_VIEW_THRESHOLD = 3;
const RECENT_DAYS_THRESHOLD = 7;

const store = useGameStore();

const activeTab = ref<'all' | 'favorite'>('all');

const tabs = [
  { label: '全部', value: 'all' as const },
  { label: '收藏', value: 'favorite' as const },
];

const speeds = [
  { label: '慢', value: 2000 },
  { label: '中', value: 1000 },
  { label: '快', value: 500 },
  { label: '极快', value: 200 },
];

const filteredRecords = computed(() => {
  if (activeTab.value === 'favorite') {
    return store.gameRecords.filter(r => r.isFavorite);
  }
  return store.gameRecords;
});

function isFrequentlyUsed(record: GameRecord): boolean {
  if (record.viewCount >= FREQUENT_VIEW_THRESHOLD) return true;
  const daysSinceLastView = (Date.now() - record.lastViewedAt) / (1000 * 60 * 60 * 24);
  return daysSinceLastView <= RECENT_DAYS_THRESHOLD;
}

function handleDelete(record: GameRecord) {
  if (record.isFavorite) {
    const ok = window.confirm('该棋谱已收藏，确定要删除吗？');
    if (!ok) return;
  } else {
    const ok = window.confirm('确定要删除这条棋谱记录吗？');
    if (!ok) return;
  }
  store.deleteRecord(record.id);
}
</script>
