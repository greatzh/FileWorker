<script setup lang="ts">
import { onBeforeMount, ref, type Ref } from 'vue';
import { formatBytes } from '@/utils/utils';
import { DeleteFile, ListFiles } from '@/api';
import type { _Object } from '@aws-sdk/client-s3';

let uploadedFiles: Ref<_Object[]> = ref([]);

const refreshFiles = async () => {
    const res = await ListFiles();
    if (res.Contents) {
        uploadedFiles.value = res.Contents;
    }
};

onBeforeMount(async () => {
    await refreshFiles();
});

const onDeleteFileClick = async (key?: string) => {
    if (!key) {
        return;
    }
    await DeleteFile(key);
    await refreshFiles();
};

// 添加解码文件名的方法
const decodeFileName = (fileName: string) => {
  return decodeURIComponent(fileName);
};

const tooltipVisible = ref(false); // 控制tooltip显示的响应式变量

// Specify the type of the parameter as string
const copyDownloadLink = async (fileKey: string) => {
  try {
    await navigator.clipboard.writeText(`${window.location.origin}/${fileKey}`);
    tooltipVisible.value = true; // 显示tooltip
    setTimeout(() => {
      tooltipVisible.value = false; // 3秒后隐藏tooltip
    }, 3000); // 3000毫秒 = 3秒
  } catch (err) {
    console.error('Failed to copy: ', err);
  }
};
</script>

<template>
    <div class="flex flex-col items-center mt-5">
        <h1 class="text-lg">{{ $t("page_title.filemanage") }}</h1>
        <div class="px-4 py-4 max-w-screen-md w-4/5">
            <div v-for="file in uploadedFiles" :key="file.Key"
                class="w-full flex flex-col items-center mt-4 rounded border-1 border-gray-300 px-2 py-1">
                <div class="flex flex-col flex-grow">
                    <a class="text-lg font-semibold file-name" :href="`/${file.Key}`" target="_blank" download>{{ decodeFileName(file.Key || '') }}</a>
                    <div class="text-sm text-gray">{{ formatBytes(file.Size ?? 0) }}</div>
                </div>
                <div class="flex">
                    <!-- 新增下载按钮 -->
                    <a class="ml-auto w-6 h-6 i-mdi-download-outline cursor-pointer"
                    :href="`/${file.Key}`" download :title="$t('common.download')"></a>
                    <div class="w-6 h-6 i-mdi-trash-can-outline cursor-pointer ml-2"
                        @click="onDeleteFileClick(file.Key)" :title="$t('common.delete')"></div>
                    <div v-if="tooltipVisible" class="fixed bottom-5 left-1/2 transform -translate-x-1/2 px-2 py-1 bg-black text-white rounded opacity-70 transition-opacity duration-300">{{ $t('alert.copy_link_success')}}</div>                    
                    <!-- 复制下载链接的按钮，并添加左边距 -->
                    <div class="w-6 h-6 i-mdi-link-variant cursor-pointer ml-2"
                        @click="copyDownloadLink(file.Key || '')" :title="$t('common.copylink')"></div>
                </div>
            </div>
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

.file-name {
    overflow-wrap: break-word;
}
</style>