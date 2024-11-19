<template>
    <div class="container">
        <div
            v-if="canUploadFile && uploadStatus !== 'success'"
            class="chat-input-button-file">
            <label>
                <input type="file" @change="onFileSelected" />
                <span>
                    <i class="pi pi-file-arrow-up label" />
                </span>
            </label>
        </div>
        <div v-if="!canUploadFile && uploadStatus === 'success'">
            <FilePreview
                :previewUrl="previewUrl"
                :isImage="isImage"
                :isVideo="isVideo"
                :fileName="selectedFile.name"
                @reset="resetSelectedFile" />
        </div>
        <div v-else-if="uploadStatus === 'uploading'">
            <p>Загрузка файла...</p>
        </div>
        <div v-else-if="uploadStatus === 'error'">
            <p>Ошибка при загрузке файла.</p>
        </div>
    </div>

    <FileDropDownMenu
        class="file-drop-down"
        :can-upload-file="canUploadFile"
        @file-selected="handleFileChange" />
</template>

<script setup>
import {ref, watch, nextTick} from 'vue'
import FileDropDownMenu from './FileDropDownMenu.vue'
import FilePreview from './FilePreview.vue'
const props = defineProps({
    canUploadFile: {
        type: Boolean,
        required: true,
        default: true,
    },
})

const clicked = ref(false)

const selectedFile = ref(null)
const uploadStatus = ref('')
const fileLink = ref('')
const previewUrl = ref('')
const isImage = ref(false)
const isVideo = ref(false)

const emit = defineEmits(['fileUploaded'])
watch(
    () => props.canUploadFile,
    () => {
        nextTick(() => {
            if (props.canUploadFile === true) {
                uploadStatus.value = null
            }
        })
    }
)

const resetSelectedFile = () => {
    selectedFile.value = null
    emit('fileUploaded', '')
    fileLink.value = ''
    previewUrl.value = ''
}

const onFileSelected = (event) => {
    clicked.value = false
    console.log('onFileSelected', event.target.files[0])
    selectedFile.value = event.target.files[0]
    if (selectedFile.value) {
        generatePreview()
        uploadFile()
    }
}

const handleFileChange = (file) => {
    console.log('onFileSelected', file)
    selectedFile.value = file
    if (selectedFile.value) {
        generatePreview()
        uploadFile()
    }
}

const generatePreview = () => {
    const file = selectedFile.value
    const fileType = file.type
    console.log(fileType)

    if (fileType.startsWith('image/')) {
        isImage.value = true
        isVideo.value = false
    } else if (fileType.startsWith('video/')) {
        isImage.value = false
        isVideo.value = true
    } else {
        isImage.value = false
        isVideo.value = false
    }

    if (isImage.value || isVideo.value) {
        const reader = new FileReader()
        reader.onload = (e) => {
            previewUrl.value = e.target.result
        }
        reader.readAsDataURL(file)
    } else {
        previewUrl.value = '' // No preview available
    }
}

const uploadFile = async () => {
    uploadStatus.value = 'uploading'
    const formData = new FormData()
    formData.append('file', selectedFile.value)

    try {
        const response = await fetch(
            'https://filebump.services.mobilon.ru/upload',
            {
                method: 'POST',
                body: formData,
            }
        )
        const result = await response.json()
        fileLink.value = result.url
        uploadStatus.value = 'success'
        props.canUploadFile = false

        // emit event with link
        emit('fileUploaded', {url: fileLink.value, type: 'message.file'})
    } catch (error) {
        console.error('Ошибка при загрузке файла:', error)
        uploadStatus.value = 'error'
    }
}
</script>

<style scoped>
.chat-input-button-file {
    input {
        position: absolute;
        z-index: -1;
        opacity: 0;
        display: block;
        width: 0;
        height: 0;
    }

    .label {
        display: block;
        font-size: 1.3rem;
        color: #525252;
        cursor: pointer;
        padding: 14px;
    }
}

.preview-image,
.preview-video {
    max-width: 200px;
    max-height: 200px;
    border-radius: 5px;
}

.container {
    align-items: center;
    justify-content: center;
    min-width: 100px;
    min-height: 100px;
    position: relative;
    display: flex;
    border-radius: 5px;
    border: 1px solid lightgray;
    background-color: #fbfbfb;
}

.file-drop-down {
    display: none;
}

.file-drop-down:hover {
    display: inherit;
}

.container:hover + .file-drop-down {
    display: inherit;
}

.preview {
    max-width: 150px;
    max-height: 100px;
    overflow: hidden;
    border-radius: 5px;
    border: 1px solid;
}

.preview-container {
    margin-left: 10px;
}

.preview-name {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    max-width: 150px;
    width: 150px;
    display: inline-block;
}
</style>
