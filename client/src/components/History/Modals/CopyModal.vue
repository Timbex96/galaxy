<script setup lang="ts">
import {
    BAlert,
    BButton,
    BForm,
    BFormGroup,
    BFormInput,
    BFormInvalidFeedback,
    BFormRadio,
    BModal,
    BSpinner,
} from "bootstrap-vue";
import { storeToRefs } from "pinia";
import { computed, ref, watch } from "vue";

import type { HistorySummary } from "@/api";
import { useHistoryStore } from "@/stores/historyStore";
import { useUserStore } from "@/stores/userStore";
import localize from "@/utils/localization";

interface Props {
    history: HistorySummary;
}

const props = defineProps<Props>();

const userStore = useUserStore();
const historyStore = useHistoryStore();

const { currentUser, isAnonymous } = storeToRefs(userStore);

const name = ref("");
const copyAll = ref(false);
const loading = ref(false);

const title = computed(() => {
    return `Copying History: ${props.history.name}`;
});
const saveTitle = computed(() => {
    return loading.value ? "Saving..." : "Copy History";
});
const saveVariant = computed(() => {
    return loading.value ? "info" : formValid.value ? "primary" : "secondary";
});
const userOwnsHistory = computed(() => {
    return userStore.isRegisteredUser(currentUser.value) && currentUser.value.id == props.history?.user_id;
});
const newNameValid = computed(() => {
    if (userOwnsHistory.value && name.value == props.history.name) {
        return false;
    }
    return name.value.length > 0;
});
const formValid = computed(() => {
    return newNameValid.value;
});

watch(
    () => props.history,
    (newHistory) => {
        name.value = `Copy of '${newHistory.name}'`;
    },
    {
        immediate: true,
    }
);

async function copy(close: () => void) {
    loading.value = true;
    await historyStore.copyHistory(props.history, name.value, copyAll.value);
    loading.value = false;
    close();
}
</script>

<template>
    <BModal v-bind="$attrs" :title="title" title-tag="h2" v-on="$listeners">
        <transition name="fade">
            <BAlert v-localize :show="isAnonymous" variant="warning">
                As an anonymous user, unless you log in or register, you will lose your current history after copying
                this history. You can <a href="/user/login">log in here</a> or <a href="/user/create">register here</a>.
            </BAlert>
        </transition>

        <transition name="fade">
            <div v-if="loading" class="d-flex justify-content-center mb-3">
                <BSpinner label="Copying History..." />
            </div>
        </transition>

        <transition>
            <BForm v-if="!loading">
                <BFormGroup label="Enter a title for the new history" label-for="copy-modal-title">
                    <BFormInput id="copy-modal-title" v-model="name" :state="newNameValid" required />

                    <BFormInvalidFeedback :state="newNameValid">
                        Please enter a valid history title.
                    </BFormInvalidFeedback>
                </BFormGroup>

                <BFormGroup label="Choose which datasets from the original history to include.">
                    <BFormRadio v-model="copyAll" :value="false">
                        Copy only the active, non-deleted datasets.
                    </BFormRadio>

                    <BFormRadio v-model="copyAll" :value="true"> Copy all datasets including deleted ones. </BFormRadio>
                </BFormGroup>
            </BForm>
        </transition>

        <template v-slot:modal-footer="{ ok, cancel }">
            <BButton class="mr-3" @click="cancel()"> Cancel </BButton>

            <BButton :variant="saveVariant" :disabled="loading || !formValid" @click="copy(ok)">
                {{ localize(saveTitle) }}
            </BButton>
        </template>
    </BModal>
</template>

<style lang="scss">
@import "scss/transitions.scss";
</style>
