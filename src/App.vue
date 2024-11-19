<script setup>
import FileUploader from './components/FileUploader.vue'
import {computed, ref, unref} from 'vue'

const fileLink = ref(null)

const canUploadFile = computed(() => {
    return !fileLink.value || fileLink.value === ''
})

const fileUploaded = (obj) => {
    console.log('fileUploaded', obj)
    fileLink.value = obj
}
</script>

<template>
    <FileUploader
        :can-upload-file="canUploadFile"
        @file-uploaded="fileUploaded" />
</template>

<style scoped></style>
