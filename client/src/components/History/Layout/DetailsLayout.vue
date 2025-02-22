<script setup lang="ts">
import { library } from "@fortawesome/fontawesome-svg-core";
import { faPen, faSave, faUndo } from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { BButton, BFormInput, BFormTextarea } from "bootstrap-vue";
import { storeToRefs } from "pinia";
import { computed, ref } from "vue";

import { useUserStore } from "@/stores/userStore";
import l from "@/utils/localization";

import StatelessTags from "@/components/TagsMultiselect/StatelessTags.vue";

library.add(faPen, faSave, faUndo);

interface Props {
    name?: string;
    tags?: string[];
    writeable?: boolean;
    annotation?: string;
    showAnnotation?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
    name: undefined,
    tags: undefined,
    writeable: true,
    annotation: undefined,
    showAnnotation: true,
});

const emit = defineEmits(["save"]);

const userStore = useUserStore();
const { isAnonymous } = storeToRefs(userStore);

const nameRef = ref<HTMLInputElement | null>(null);

const editing = ref(false);
const textSelected = ref(false);
const localProps = ref<{ name: string; annotation: string | null; tags: string[] }>({
    name: "",
    annotation: "",
    tags: [],
});

const editButtonTitle = computed(() => {
    if (isAnonymous.value) {
        return l("Log in to Rename History");
    } else {
        if (props.writeable) {
            return l("Edit");
        } else {
            return l("Not Editable");
        }
    }
});

function onSave() {
    editing.value = false;
    emit("save", localProps.value);
}

function onToggle() {
    editing.value = !editing.value;

    localProps.value = {
        name: props.name,
        annotation: props.annotation,
        tags: props.tags,
    };

    if (nameRef.value) {
        nameRef.value.focus();
    }
}

function selectText() {
    if (!textSelected.value) {
        nameRef.value?.select();
        textSelected.value = true;
    } else {
        nameRef.value?.focus();
        textSelected.value = false;
    }
}
</script>

<template>
    <section class="m-3 details" data-description="edit details">
        <BButton
            :disabled="isAnonymous || !writeable"
            class="edit-button ml-1 float-right"
            data-description="editor toggle"
            size="sm"
            variant="link"
            :title="editButtonTitle"
            :pressed="editing"
            @click="onToggle">
            <FontAwesomeIcon :icon="faPen" fixed-width />
        </BButton>

        <slot name="name" />

        <div v-if="!editing">
            <div v-if="annotation" v-short="annotation" class="mt-2" data-description="annotation value" />

            <StatelessTags v-if="tags" class="tags mt-2" :value="tags" :disabled="true" />
        </div>

        <div v-else class="mt-3" data-description="edit form">
            <BFormInput
                ref="name"
                v-model="localProps.name"
                class="mb-2"
                placeholder="Name"
                trim
                max-rows="4"
                data-description="name input"
                @keyup.enter="onSave"
                @keyup.esc="onToggle"
                @focus="selectText"
                @blur="textSelected = false" />

            <BFormTextarea
                v-if="showAnnotation"
                v-model="localProps.annotation"
                class="mb-2"
                placeholder="Annotation (optional)"
                trim
                max-rows="4"
                data-description="annotation input"
                @keyup.esc="onToggle" />

            <StatelessTags v-if="localProps.tags" v-model="localProps.tags" class="mb-3 tags" />

            <BButton
                class="save-button mb-1"
                data-description="editor save button"
                size="sm"
                variant="primary"
                :disabled="!localProps.name"
                @click="onSave">
                <FontAwesomeIcon :icon="faSave" fixed-width />
                <span v-localize>Save</span>
            </BButton>

            <BButton
                class="cancel-button mb-1"
                data-description="editor cancel button"
                size="sm"
                icon="undo"
                @click="onToggle">
                <FontAwesomeIcon :icon="faUndo" fixed-width />
                <span v-localize>Cancel</span>
            </BButton>
        </div>
    </section>
</template>
