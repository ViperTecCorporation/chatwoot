<script setup>
import { computed } from 'vue';
import { useKeyboardEvents } from 'dashboard/composables/useKeyboardEvents';
import wootConstants from 'dashboard/constants/globals';

const props = defineProps({
  items: {
    type: Array,
    default: () => [],
  },
  activeTab: {
    type: String,
    default: wootConstants.ASSIGNEE_TYPE.ME,
  },
});

const emit = defineEmits(['chatTabChange']);

const HORIZONTAL_TAB_ORDER = [
  wootConstants.ASSIGNEE_TYPE.ME,
  wootConstants.ASSIGNEE_TYPE.WAITING,
  wootConstants.ASSIGNEE_TYPE.UNASSIGNED,
  wootConstants.ASSIGNEE_TYPE.ANSWERED,
  wootConstants.ASSIGNEE_TYPE.MENTION,
  wootConstants.ASSIGNEE_TYPE.PARTICIPATING,
  wootConstants.ASSIGNEE_TYPE.ALL,
  wootConstants.ASSIGNEE_TYPE.GROUPS,
  wootConstants.ASSIGNEE_TYPE.INTERNAL,
];

const sortedItems = computed(() => {
  if (!props.items || !Array.isArray(props.items)) {
    return [];
  }
  const itemsMap = new Map(props.items.map(item => [item.key, item]));
  return HORIZONTAL_TAB_ORDER
    .filter(key => itemsMap.has(key))
    .map(key => itemsMap.get(key));
});

const activeTabIndex = computed(() => {
  return sortedItems.value.findIndex(item => item.key === props.activeTab);
});

const onTabChange = selectedTabIndex => {
  if (selectedTabIndex >= 0 && selectedTabIndex < sortedItems.value.length) {
    const selectedItem = sortedItems.value[selectedTabIndex];
    if (selectedItem.key !== props.activeTab) {
      emit('chatTabChange', selectedItem.key);
    }
  }
};

const keyboardEvents = {
  'Alt+KeyN': {
    action: () => {
      if (props.activeTab === wootConstants.ASSIGNEE_TYPE.ALL) {
        onTabChange(0);
      } else {
        const nextIndex = (activeTabIndex.value + 1) % sortedItems.value.length;
        onTabChange(nextIndex);
      }
    },
  },
};

useKeyboardEvents(keyboardEvents);
</script>

<template>
  <div class="flex items-center gap-1 px-3 py-2 border-b border-n-strong overflow-x-auto">
    <button
      v-for="(item, index) in sortedItems"
      :key="item.key"
      class="px-3 py-1.5 text-sm font-medium rounded-md whitespace-nowrap transition-colors duration-150"
      :class="
        activeTab === item.key
          ? 'bg-n-brand-9 text-white'
          : 'text-n-slate-11 hover:bg-n-slate-3 hover:text-n-slate-12'
      "
      @click="onTabChange(index)"
    >
      {{ item.name }}
      <span
        v-if="item.count > 0"
        class="ml-1.5 px-1.5 py-0.5 text-xs rounded-full"
        :class="
          activeTab === item.key
            ? 'bg-white/20 text-white'
            : 'bg-n-slate-4 text-n-slate-11'
        "
      >
        {{ item.count }}
      </span>
    </button>
  </div>
</template>
