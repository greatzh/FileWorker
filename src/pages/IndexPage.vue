<script setup lang="ts">
import { useI18n } from "vue-i18n";
import { useRouter } from "vue-router";
import useI18nStore from "../store/i18n";

const router = useRouter();

const i18nStore = useI18nStore();
let updateLocale = (locale: string) => {
  i18nStore.setLocale(locale);
};

const { locale } = useI18n();
if (i18nStore.locale !== "") {
  locale.value = i18nStore.locale;
}

let onClipAreaClick = () => {
  router.push("/clip");
};

let onUploadClick = () => {
  router.push("/file");
};
</script>

<template>
  <div class="items-center max-w-full my-[15vw] mx-auto px-[8vw]">
    <div class="cards-grid">
      <div class="pannel file-pannel">
        <div class="text-2xl">
          <router-link to="/filemanage" class="link-hint">{{ $t("index.file_channel_title") }}</router-link>
        </div>
        <div class="upload-area hover:border-gray-900" @click="onUploadClick">
          <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 384 512">
            <path
              d="M320 464c8.8 0 16-7.2 16-16V160H256c-17.7 0-32-14.3-32-32V48H64c-8.8 0-16 7.2-16 16V448c0 8.8 7.2 16 16 16H320zM0 64C0 28.7 28.7 0 64 0H229.5c17 0 33.3 6.7 45.3 18.7l90.5 90.5c12 12 18.7 28.3 18.7 45.3V448c0 35.3-28.7 64-64 64H64c-35.3 0-64-28.7-64-64V64z" />
          </svg>
          <p class="text-xl mt-4">{{ $t("index.clicktoenter") }}</p>
        </div>
        <p><span>{{ $t("common.or") }}</span><router-link to="/filemanage" class="text-blue-600 hover:underline">{{
          $t("index.file_manager") }}</router-link></p>
      </div>
      <div class="pannel clip-pannel">
        <div class="text-2xl flex flex-row items-center">
          <router-link to="/clipmanage" class="link-hint">{{ $t("index.clip_channel_title") }}</router-link>
        </div>
        <div class="clip-area hover:border-gray-900" @click="onClipAreaClick">
          <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 512 512">
            <path
              d="M441 58.9L453.1 71c9.4 9.4 9.4 24.6 0 33.9L424 134.1 377.9 88 407 58.9c9.4-9.4 24.6-9.4 33.9 0zM209.8 256.2L344 121.9 390.1 168 255.8 302.2c-2.9 2.9-6.5 5-10.4 6.1l-58.5 16.7 16.7-58.5c1.1-3.9 3.2-7.5 6.1-10.4zM373.1 25L175.8 222.2c-8.7 8.7-15 19.4-18.3 31.1l-28.6 100c-2.4 8.4-.1 17.4 6.1 23.6s15.2 8.5 23.6 6.1l100-28.6c11.8-3.4 22.5-9.7 31.1-18.3L487 138.9c28.1-28.1 28.1-73.7 0-101.8L474.9 25C446.8-3.1 401.2-3.1 373.1 25zM88 64C39.4 64 0 103.4 0 152V424c0 48.6 39.4 88 88 88H360c48.6 0 88-39.4 88-88V312c0-13.3-10.7-24-24-24s-24 10.7-24 24V424c0 22.1-17.9 40-40 40H88c-22.1 0-40-17.9-40-40V152c0-22.1 17.9-40 40-40H200c13.3 0 24-10.7 24-24s-10.7-24-24-24H88z" />
          </svg>
          <p class="text-xl mt-4">{{ $t("index.clicktoenter") }}</p>
        </div>
        <p><span>{{ $t("common.or") }}</span><router-link to="/clipmanage" class="text-blue-600 hover:underline">{{
          $t("index.clip_manager") }}</router-link></p>
      </div>
      <!-- <div class="pannel tips-pannel">{{ $t("index.tips_content") }}</div> -->
    </div>
    <select v-model="$i18n.locale" class="border border-gray-400 rounded-md p-1 items-center my-8 mx-auto flex"
      @change="updateLocale($i18n.locale)">
      <option v-for="locale in $i18n.availableLocales" :key="`locale-${locale}`" :value="locale">{{ locale }}</option>
    </select>
  </div>
</template>

<style scoped>
.cards-grid {
  grid-gap: 2rem;
  align-items: center;
  display: grid;
  gap: 2rem;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
}

.pannel {
  --uno: block p-4 mx-auto rounded w-full;
}

/* .file-pannel {
  background-color: #f0f0f0;
}

.clip-pannel {
  background-color: #f0f0f0;
} */

.tips-pannel {
  background-color: #d1e7dd;
}

.upload-area,
.clip-area {
  --uno: flex flex-col justify-center items-center h-42 border-solid border-1 border-gray-300 mt-2 p-8 cursor-pointer rounded-lg;
}

.link-hint {
  text-decoration: underline;
  text-decoration-style: dashed;
  text-decoration-color: #a0aec0;
  text-underline-offset: 0.2em;
}

.link-hint:hover {
  text-decoration-color: black;
}
</style>

<style>
html,
body,
#app {
  margin: 0;
  padding: 0;
  background-color: #f8f9fa;
}
</style>