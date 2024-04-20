<script setup lang="ts">
import { onMounted, onUnmounted, ref, type Ref } from 'vue';
import useFileStore from '@/store/file';
import { formatBytes } from '@/utils/utils';
import { PutFile } from '@/api';

const fileStore = useFileStore();

let fileUploadInput = ref();

let requestUploadFile = () => {
  fileUploadInput.value.click();
}

interface UploadedFile {
  name: string;
  size: number;
  visibility: string;
  done: boolean;
}

let uploadedFiles: Ref<UploadedFile[]> = ref([]);

const uploadSingle = async (index: number, filename: string, file: File) => {
  await PutFile(filename, file, fileStore.visibility, "file");
  uploadedFiles.value[index - 1].done = true;
}

onMounted(() => {
  fileUploadInput.value.addEventListener('change', async (event: Event) => {
    const target = event.target as HTMLInputElement;
    const { files } = target;
    if (files && files.length > 0) {
      for (let i = 0; i < files.length; i++) {
        const file = files[i];
        const index = uploadedFiles.value.push({
          name: file.name,
          size: file.size,
          visibility: fileStore.visibility,
          done: false
        });
        try {
          uploadSingle(index, file.name, file);
        } catch (error) {
          console.error(error);
        }
      }
    }
  });
});

let fileUploadArea = ref();

const onDragEvent = async (event: DragEvent) => {
  event.preventDefault();
  event.stopPropagation();
  if (event.type === 'dragover') {
    fileUploadArea.value.style.border = '6px dashed #d0d7de';
  } else {
    fileUploadArea.value.style.border = '6px dashed #e5e7eb';
  }
  if (event.type === 'drop') {
    const files = event.dataTransfer?.files;
    if (files && files.length > 0) {
      for (let i = 0; i < files.length; i++) {
        const file = files[i];
        const index = uploadedFiles.value.push({
          name: file.name,
          size: file.size,
          visibility: fileStore.visibility,
          done: false
        });
        try {
          uploadSingle(index, file.name, file);
        } catch (error) {
          console.error(error);
        }
      }
    }
  }
}
onMounted(() => {
  fileUploadArea.value.addEventListener('dragenter', onDragEvent);
  fileUploadArea.value.addEventListener('dragover', onDragEvent);
  fileUploadArea.value.addEventListener('dragleave', onDragEvent);
  fileUploadArea.value.addEventListener('drop', onDragEvent);
});
onUnmounted(() => {
  fileUploadArea.value.removeEventListener('dragenter', onDragEvent);
  fileUploadArea.value.removeEventListener('dragover', onDragEvent);
  fileUploadArea.value.removeEventListener('dragleave', onDragEvent);
  fileUploadArea.value.removeEventListener('drop', onDragEvent);
});

</script>

<template>
  <div class="items-center max-w-full my-[15vw] mx-auto px-[8vw]">
    <div class="file-area flex flex-col mt-8 mx-auto">
      <div class="files hover:border-gray-900" @click="requestUploadFile" ref="fileUploadArea">
        <input ref="fileUploadInput" type="file" class="hidden" multiple />
        <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 384 512"><path d="M320 464c8.8 0 16-7.2 16-16V160H256c-17.7 0-32-14.3-32-32V48H64c-8.8 0-16 7.2-16 16V448c0 8.8 7.2 16 16 16H320zM0 64C0 28.7 28.7 0 64 0H229.5c17 0 33.3 6.7 45.3 18.7l90.5 90.5c12 12 18.7 28.3 18.7 45.3V448c0 35.3-28.7 64-64 64H64c-35.3 0-64-28.7-64-64V64z"/></svg>
        <p class="text-xl">{{ $t("index.clicktoupload") }}</p>
      </div>
      <div class="footer p-2 mx-auto rounded-md">
        <select class="public-select" v-model="fileStore.visibility">
          <option value="private">{{ $t('common.private') }}</option>
          <option value="public">{{ $t('common.public') }}</option>
        </select>
      </div>
    </div>
    <div class="px-4 py-4 max-w-screen-md w-11/12 mx-auto">
      <a v-for="file in uploadedFiles" :key="file.name" class="w-full flex flex-row items-center mt-4"
        :href="`/${file.name}`" target="_blank">
        <div class="w-10 h-10 i-mdi-file-document-outline"></div>
        <div class="flex flex-col">
          <div class="text-lg font-semibold">{{ file.name }}</div>
          <div class="text-sm text-gray">{{ formatBytes(file.size) }} {{ file.visibility }}</div>
        </div>
        <div class="ml-auto w-6 h-6" :class="file.done ? 'i-mdi-check' : 'uploading'"></div>
      </a>
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

.file-area .footer .public-select {
  --uno: border-1 rounded-md p-2 text-sm;
  border-color: #d1d1d1;
  outline-color: #0969da;
}

.file-area .footer .save-btn {
  --uno: rounded-md px-6 py-1.5 text-sm ml-auto text-white;
  background-color: #1f883d;
}

.file-area .footer .save-btn:hover {
  background-color: #1a7f37;
}

.file-area .header {
  --uno: border-1 rounded px-3 py-2 text-sm w-60;
  border-color: #d1d1d1;
  outline-color: #0969da;
}

.files {
  --uno: flex flex-col justify-center items-center h-64 border-solid border-1 border-gray-300 mt-2 p-8 cursor-pointer rounded-lg;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}

.uploading {
  border: 5px solid #f3f3f3;
  border-top: 5px solid #555;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: inline-block;
  animation: spin 2s linear infinite;
}
</style>
