<script setup lang="ts">
import { minimalSetup } from "codemirror"
import { EditorState } from "@codemirror/state"
import { EditorView, lineNumbers, highlightSpecialChars, drawSelection, dropCursor, keymap } from "@codemirror/view"
// 确保导入lineWrapping
import { defaultKeymap } from "@codemirror/commands"

import { onMounted, onBeforeUnmount, ref } from "vue";
import useClipStore from "@/store/clip";

import { PutFile } from "@/api";
import { getRandomFilename } from "@/utils/utils";
import { toast } from '@/utils/toast'
import { useI18n } from 'vue-i18n';

const code = ref("");
const modified = ref(false);
const editorElement = ref();
const { t: $t } = useI18n();
let editor: EditorView;

let startState = EditorState.create({
  doc: "",
  extensions: [
    minimalSetup,
    lineNumbers(),
    highlightSpecialChars(),
    drawSelection(),
    dropCursor(),
    EditorView.lineWrapping, // 添加这行来启用文本自动换行
    keymap.of(defaultKeymap),
    EditorView.updateListener.of((update) => {
      code.value = update.state.doc.toString();
      if (update.docChanged) {
        modified.value = true;
      }
    }),
  ]
})

onMounted(() => {
  editor = new EditorView({
    state: startState,
    parent: editorElement.value,
  })
  editor.requestMeasure({
    read: () => {
      editor.focus();
    }
  })
})

let filename = ref(getRandomFilename());

let refreshRandomFileName = () => {
  filename.value = getRandomFilename();
}

const clipStore = useClipStore();

let onSaveBtnClick = async () => {
  await PutFile(filename.value, code.value, clipStore.visibility, "text");
  modified.value = false;
  toast($t("index.save_success"), 'success');
}

let saveContentKeydown = (e: KeyboardEvent) => {
  if ((e.ctrlKey && e.key === "s") || (e.metaKey && e.key === "s")) {
    e.preventDefault();
    onSaveBtnClick();
  }
}

let onPasteFile = async (e: ClipboardEvent) => {
  if (!e.clipboardData?.files.length) {
    return;
  }
  const file = e.clipboardData.files[0];
  console.log(file);
  const text = await file.text();
  const cursor = editor.state.selection.main.head;
  editor.dispatch({
    changes: { from: cursor, insert: text },
  });
}


onMounted(() => {
  window.addEventListener("keydown", saveContentKeydown);
  document.addEventListener("paste", onPasteFile);
})

onBeforeUnmount(() => {
  window.removeEventListener("keydown", saveContentKeydown);
  document.removeEventListener("paste", onPasteFile);
})

</script>

<template>
  <div class="items-center max-w-full my-[8vw] mx-auto px-[8vw]">
    <div class="flex flex-row p-2 items-center">
      <input class="filename-input monospace" type="text" v-model="filename" :placeholder="$t('common.filename')" />
      <button @click="refreshRandomFileName" class="i-mdi-refresh ml-1 w-5 h-5"></button>
      <div :class="modified ? 'unsave-attention' : 'save-attention'"></div>
    </div>
    <div class="text-area flex flex-col border-gray-900">
      <div ref="editorElement"></div>
    </div>
    <div class="p-2 flex flex-row items-center mx-auto justify-center">
      <select class="public-select" v-model="clipStore.visibility">
        <option value="private">{{ $t('common.private') }}</option>
        <option value="public">{{ $t('common.public') }}</option>
      </select>
      <button class="save-btn" @click="onSaveBtnClick">{{ $t('common.save') }}</button>
    </div>
  </div>
</template>

<style>
html,
body,
#app {
  margin: 0;
  padding: 0;
  background-color: #f8f9fa;
}


.tips-pannel {
  background-color: #d1e7dd;
}

.text-area {
  --uno: rounded-lg max-w-full border-1 border-gray-300;
}

.public-select {
  --uno: border-1 rounded-lg px-2 py-1.5 text-sm;
  border-color: #d1d1d1;
  outline-color: #0969da;
}

.save-btn {
  --uno: rounded-lg px-6 py-1.5 text-sm ml-5 text-white;
  background-color: #434343;
}

.save-btn:hover {
  background-color: #000000;
}

.filename-input {
  --uno: border-1 rounded-lg p-2 text-sm w-40;
  border-color: #d1d1d1;
  outline-color: #0969da;
}

.cm-editor {
  height: 500px;
  font-size: large;
  padding: 8px;
}

.cm-editor.cm-focused {
  outline: none;
}

.cm-gutter.cm-lineNumbers {
  background-color: #f8f9fa;
  border-radius: 8px;
}

.cm-gutters {
  border: none !important;
}

.cm-selectionBackground {
  background-color: #54aeff66 !important;
}

.unsave-attention {
  --uno: i-mdi-circle-small w-8 h-8 ml-auto;
  color: #9a6700 !important;
}

.save-attention {
  --uno: i-mdi-circle-small w-8 h-8 ml-auto;
  color: #1f883d !important;
}
</style>
