<script setup lang="ts">
import { onBeforeMount, ref, type Ref } from 'vue';
import { formatBytes } from '@/utils/utils';
import { DeleteFile, ListFiles } from '@/api';
import type { _Object } from '@aws-sdk/client-s3';
import { toast } from '@/utils/toast';
import { useI18n } from 'vue-i18n';

const { t: $t } = useI18n();
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

// Specify the type of the parameter as string
const copyDownloadLink = async (fileKey: string) => {
    try {
        await navigator.clipboard.writeText(`${window.location.origin}/${fileKey}`);
        toast($t('alert.copy_link_success'), 'success'); // 使用toast显示成功消息
    } catch (err) {
        console.error('Failed to copy: ', err);
        toast($t('alert.copy_link_failed'), 'error'); // 使用toast显示失败消息
    }
};
</script>

<template>
    <div class="items-center max-w-full my-[15vw] mx-auto px-[8vw]">
        <h1 class="text-2xl">{{ $t("page_title.filemanage") }}</h1>
        <div class="p-2 w-11/12">
            <div v-for="file in uploadedFiles" :key="file.Key"
                class="w-full flex flex-col mt-4 rounded-lg border-1 border-gray-300 p-2 hover:border-gray-900">
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