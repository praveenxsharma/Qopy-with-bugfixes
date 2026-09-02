<template>
  <div class="topbar">
    <input
      ref="searchInput"
      v-model="searchQuery"
      @input="onInputChange"
      class="search"
      autocorrect="off"
      autocapitalize="off"
      spellcheck="false"
      type="text"
      placeholder="Type to filter entries..." />
    <div class="filters">
      <button
        v-for="f in filterOptions"
        :key="f.value"
        class="filter-chip"
        :class="{ active: activeFilter === f.value }"
        @click="setFilter(f.value)">
        {{ f.label }}
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";

const searchQuery = ref("");
const searchInput = ref<HTMLInputElement | null>(null);
const activeFilter = ref<string>("all");

const filterOptions = [
  { label: "All", value: "all" },
  { label: "Text", value: "text" },
  { label: "Link", value: "link" },
  { label: "Image", value: "image" },
  { label: "File", value: "file" },
  { label: "Color", value: "color" },
  { label: "Code", value: "code" },
];

const emit = defineEmits<{
  (e: "search", query: string): void;
  (e: "searchStarted"): void;
  (e: "focus"): void;
  (e: "filter", filter: string): void;
}>();

const onInputChange = () => {
  emit("searchStarted");
  emit("search", searchQuery.value);
};

const setFilter = (value: string) => {
  activeFilter.value = value;
  emit("filter", value);
};

defineExpose({ searchInput });
</script>

<style lang="scss">
.topbar {
  width: 100%;
  min-height: 56px;
  border-bottom: 1px solid var(--border);
  display: flex;
  align-items: center;
  padding-inline: 16px;
  z-index: 100;

  .search {
    width: 100%;
    height: 100%;
    font-size: 18px;
    color: var(--text);
    background-color: transparent;
    outline: none;
    border: none;
    font-family: SFRoundedMedium;
  }

  .filters {
    display: flex;
    gap: 6px;
    align-items: center;
    margin-left: 12px;
    flex-shrink: 0;

    .filter-chip {
      border: 1px solid var(--border);
      background: transparent;
      color: var(--text-secondary);
      font-family: SFRoundedRegular;
      font-size: 12px;
      padding: 4px 10px;
      border-radius: 999px;
      cursor: pointer;
      transition: all 0.15s;

      &:hover {
        background: var(--border);
        color: var(--text);
      }

      &.active {
        background: var(--accent);
        color: #1e1c1a;
        border-color: var(--accent);
        font-family: SFRoundedMedium;
      }
    }
  }
}
</style>
