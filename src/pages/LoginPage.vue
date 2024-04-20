<script setup lang="ts">
import { ref } from 'vue';
import Cookies from 'js-cookie'
import { toast } from '@/utils/toast';
import { useRouter } from 'vue-router';
import { useI18n } from 'vue-i18n';

const router = useRouter();
const { t: $t } = useI18n();

const password = ref('');
const onSubmitBtnClick = () => {
    Cookies.set('PASSWORD', password.value);
    toast($t("login.setting_success"), 'success');
    setTimeout(() => {
        if (window.history.state.back) {
            router.back();
        } else {
            router.replace({ path: '/' })
        }
    }, 1000);
}
</script>

<template>
    <div class="block justify-center max-w-full my-[15vw] mx-auto px-[8vw] break-all">
        <div id="board">
            <h1 class="text-3xl">{{ $t("login.login_title") }}</h1>
            <input class="h-8 my-5 px-2 py-5 w-64 rounded-lg bg-gray-100" type="password" v-model="password"
                :placeholder="$t('login.password_placeholder')">
            <button id="submit-button" class="btn" @click="onSubmitBtnClick">{{ $t("login.login_button") }}</button>
        </div>
    </div>
</template>

<style scoped>
#board {
    @apply flex flex-col p-4 items-center dark:(bg-dark-100 border-dark-300 text-gray-400);
}

.btn {
    @apply rounded-lg my-3 cursor-pointer outline-none bg-gray-600 text-white text-lg py-1 px-4 transition w-64 hover:bg-gray-900;
}
</style>