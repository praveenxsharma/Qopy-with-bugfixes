<template>
  <Teleport to="body">
    <Transition name="menu">
      <div v-if="visible" class="actions-menu" :style="menuStyle" @contextmenu.prevent>
        <p class="menu-title">Actions</p>

        <button class="menu-item" @click="action(pin, 'toggle')">
          <span class="item-icon">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M12 2L14.4 8.2L21 9.3L16.5 13.7L17.6 20.3L12 17L6.4 20.3L7.5 13.7L3 9.3L9.6 8.2L12 2Z"
                stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </span>
          <span class="item-label">{{ selectedItem?.pinned ? "Unpin" : "Pin" }}</span>
        </button>

        <button class="menu-item" @click="action(copy)">
          <span class="item-icon">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <rect x="8" y="8" width="12" height="12" rx="2" stroke="currentColor" stroke-width="1.5"/>
              <path d="M16 8V6C16 4.9 15.1 4 14 4H6C4.9 4 4 4.9 4 6V14C4 15.1 4.9 16 6 16H8" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            </svg>
          </span>
          <span class="item-label">Copy</span>
        </button>

        <button
          v-if="selectedItem && (selectedItem.content_type === 'link' || selectedItem.content_type === 'text')"
          class="menu-item"
          @click="action(openInBrowser)">
          <span class="item-icon">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <circle cx="12" cy="12" r="9" stroke="currentColor" stroke-width="1.5"/>
              <path d="M3 12H21M12 3C14.5 4.5 16 8 16 12C16 16 14.5 19.5 12 21C9.5 19.5 8 16 8 12C8 8 9.5 4.5 12 3Z" stroke="currentColor" stroke-width="1.5"/>
            </svg>
          </span>
          <span class="item-label">Open in Browser</span>
        </button>

        <button
          v-if="selectedItem && selectedItem.content_type === 'file'"
          class="menu-item"
          @click="action(openFile)">
          <span class="item-icon">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M4 20L20 4M4 20H18M4 20V6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </span>
          <span class="item-label">Open File</span>
        </button>

        <button
          v-if="selectedItem && selectedItem.content_type === 'color'"
          class="menu-item"
          @click="action(copyAsRgb)">
          <span class="item-label copy-color">Copy as RGB</span>
        </button>
        <button
          v-if="selectedItem && selectedItem.content_type === 'color'"
          class="menu-item"
          @click="action(copyAsHsl)">
          <span class="item-label copy-color">Copy as HSL</span>
        </button>

        <div class="menu-divider"></div>

        <button class="menu-item" @click="startRename">
          <span class="item-icon">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M3 17.25V21H6.75L17.81 9.94L14.06 6.19L3 17.25ZM20.71 7.04C21.1 6.65 21.1 6.02 20.71 5.63L18.37 3.29C17.98 2.9 17.35 2.9 16.96 3.29L15.13 5.12L18.88 8.87L20.71 7.04Z"
                fill="currentColor"/>
            </svg>
          </span>
          <span class="item-label">Rename</span>
        </button>

        <div v-if="renaming" class="rename-box">
          <input
            v-model="renameValue"
            class="rename-input"
            placeholder="Custom title"
            @keydown.enter.prevent="saveRename"
            @keydown.esc.prevent="cancelRename" />
          <button class="rename-save" @click="saveRename">Save</button>
        </div>

        <button class="menu-item delete" @click="action(deleteEntry)">
          <span class="item-label">Delete Entry</span>
        </button>

        <button class="menu-item delete" @click="action(clearAll)">
          <span class="item-label">Delete All Entries</span>
        </button>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup lang="ts">
import { computed, ref } from "vue";
import { invoke } from "@tauri-apps/api/core";
import type { HistoryItem } from "~/types/types";

const props = defineProps<{
  selectedItem: HistoryItem | null;
}>();

const emit = defineEmits<{
  (e: "close"): void;
  (e: "changed"): void;
}>();

const visible = ref(false);
const x = ref(0);
const y = ref(0);
const renaming = ref(false);
const renameValue = ref("");
const menuStyle = computed(() => ({
  left: `${x.value}px`,
  top: `${y.value}px`,
}));

const getContent = async (item: HistoryItem): Promise<{ content: string; type: string }> => {
  if (item.content_type === "image") {
    const base64 = await invoke<string>("read_image", { filename: item.content });
    return { content: base64, type: "image" };
  }
  return { content: item.content, type: item.content_type };
};

const copy = async (item: HistoryItem) => {
  const { content, type } = await getContent(item);
  await invoke("copy_to_clipboard", { content, contentType: type });
};

const openInBrowser = async (item: HistoryItem) => {
  await invoke("open_in_browser", { url: item.content });
};

const openFile = async (item: HistoryItem) => {
  await invoke("open_path", { path: item.content });
};

const pin = async (item: HistoryItem) => {
  await invoke<boolean>("toggle_pin_history_item", { id: item.id });
};

const deleteEntry = async (item: HistoryItem) => {
  await invoke("delete_history_item", { id: item.id });
};

const startRename = () => {
  renameValue.value = props.selectedItem?.title ?? "";
  renaming.value = true;
};

const saveRename = async () => {
  if (!props.selectedItem) return;
  await invoke("set_history_item_title", {
    id: props.selectedItem.id,
    title: renameValue.value === "" ? null : renameValue.value,
  });
  renaming.value = false;
  emit("changed");
  visible.value = false;
  emit("close");
};

const cancelRename = () => {
  renaming.value = false;
};

const clearAll = async () => {
  await invoke("clear_history");
};

const copyAsRgb = async (item: HistoryItem) => {
  const hex = item.content;
  const r = parseInt(hex.slice(1, 3), 16);
  const g = parseInt(hex.slice(3, 5), 16);
  const b = parseInt(hex.slice(5, 7), 16);
  await invoke("copy_to_clipboard", {
    content: `rgb(${r}, ${g}, ${b})`,
    contentType: "color",
  });
};

const copyAsHsl = async (item: HistoryItem) => {
  const hex = item.content;
  const r = parseInt(hex.slice(1, 3), 16) / 255;
  const g = parseInt(hex.slice(3, 5), 16) / 255;
  const b = parseInt(hex.slice(5, 7), 16) / 255;
  const max = Math.max(r, g, b);
  const min = Math.min(r, g, b);
  let h = 0;
  let s = 0;
  const l = (max + min) / 2;
  if (max !== min) {
    const d = max - min;
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
    switch (max) {
      case r:
        h = (g - b) / d + (g < b ? 6 : 0);
        break;
      case g:
        h = (b - r) / d + 2;
        break;
      case b:
        h = (r - g) / d + 4;
        break;
    }
    h *= 60;
  }
  await invoke("copy_to_clipboard", {
    content: `hsl(${Math.round(h)}, ${Math.round(s * 100)}%, ${Math.round(l * 100)}%)`,
    contentType: "color",
  });
};

const action = async (fn: (item: HistoryItem) => Promise<void>) => {
  if (fn === clearAll) {
    await clearAll();
  } else if (props.selectedItem) {
    await fn(props.selectedItem);
  }
  emit("changed");
  visible.value = false;
  emit("close");
};

const show = (clientX: number, clientY: number) => {
  x.value = clientX;
  y.value = clientY;
  visible.value = true;
};

const hide = () => {
  visible.value = false;
  emit("close");
};

defineExpose({ show, hide, visible });
</script>

<style scoped lang="scss">
.actions-menu {
  position: fixed;
  z-index: 1000;
  width: 180px;
  background: #3a3836;
  border: 1px solid var(--border);
  border-radius: 10px;
  padding: 6px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
  transform: translateX(-50%);

  .menu-title {
    font-family: SFRoundedSemiBold;
    font-size: 11px;
    color: var(--text-muted);
    padding: 4px 8px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }

  .menu-item {
    display: flex;
    align-items: center;
    gap: 9px;
    width: 100%;
    padding: 7px 8px;
    border: none;
    background: transparent;
    border-radius: 6px;
    color: var(--text);
    font-family: SFRoundedRegular;
    font-size: 13px;
    cursor: pointer;
    text-align: left;

    &:hover {
      background: var(--border);
    }

    .item-icon {
      display: flex;
      align-items: center;
      color: var(--text-secondary);
    }

    .copy-color {
      padding-left: 25px;
    }
  }

  .menu-divider {
    height: 1px;
    background: var(--border);
    margin: 5px 0;
  }

  .rename-box {
    display: flex;
    gap: 6px;
    padding: 6px 8px;

    .rename-input {
      flex: 1;
      background: #2c2a28;
      border: 1px solid var(--border);
      border-radius: 6px;
      color: var(--text);
      font-family: SFRoundedRegular;
      font-size: 12px;
      padding: 5px 8px;
      outline: none;

      &:focus {
        border-color: var(--accent);
      }
    }

    .rename-save {
      background: var(--accent);
      color: #1e1c1a;
      border: none;
      border-radius: 6px;
      padding: 5px 10px;
      font-family: SFRoundedMedium;
      font-size: 12px;
      cursor: pointer;
    }
  }

  .menu-item.delete {
    color: #e06666;
  }
}

.menu-enter-active,
.menu-leave-active {
  transition: opacity 0.12s ease, transform 0.12s ease;
}

.menu-enter-from,
.menu-leave-to {
  opacity: 0;
  transform: translateX(-50%) scale(0.95);
}
</style>
