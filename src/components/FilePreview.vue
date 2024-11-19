<template>
    <div class="preview-container">
        <div class="preview">
            <div v-if="props.isImage">
                <img
                    :src="previewUrl"
                    alt="Image Preview"
                    class="preview-image" />
            </div>
            <div v-else-if="props.isVideo">
                <video :src="previewUrl" class="preview-video" />
            </div>
            <div v-else>
                <span class="pi pi-file" style="font-size: 2.5rem" />
            </div>
            <div class="preview-icon preview-reset" @click="handleReset">
                <span class="pi pi-times" style="font-size: 2.5rem" />
            </div>
        </div>
        <span class="preview-name">{{ fileName }}</span>
    </div>
</template>

<script setup>
import {ref, watch, nextTick} from 'vue'
const props = defineProps({
    previewUrl: {
        type: String,
        required: true,
    },
    isImage: {
        type: Boolean,
        default: false,
    },
    isVideo: {
        type: Boolean,
        default: false,
    },
    fileName: {
        type: String,
        default: '',
    },
})

const emit = defineEmits(['reset'])

function handleReset() {
    emit('reset')
}
</script>

<style scoped>
.preview-container {
    margin: 10px;
}

.preview:hover .preview-reset {
    display: flex;
}

.preview-image,
.preview-video {
    max-width: 200px;
    max-height: 200px;
    border-radius: 5px;
}

.preview {
    max-width: 150px;
    max-height: 100px;
    min-width: 150px;
    min-height: 100px;
    overflow: hidden;
    border-radius: 5px;
    border: 1px solid;
    display: flex;
    align-items: center;
    justify-content: center;
}

.preview-name {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    max-width: 150px;
    width: 150px;
    display: inline-block;
}

.preview-reset {
    display: none;
}

.preview-icon {
    margin: auto;

    position: absolute;
    background-color: white;
    opacity: 0.8;
    min-width: 150px;
    min-height: 100px;
    border-radius: 5px;
    align-items: center;
    justify-content: center;
}
</style>
