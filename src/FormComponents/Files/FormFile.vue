<template>
    <div class="existing-files row">
        <div class="file" v-for="file in files">
            <div class="file-icon">
                <div v-if="previewIcon(file)" class="thumbnail">
                    <img :src="previewIcon(file)" width="80px" />
                </div>
                <font-awesome-icon :icon="fileIcon" size="4x" v-else></font-awesome-icon>
            </div>
            <div class="action-row">
                <font-awesome-icon :icon="downloadIcon" @click="downloadFile(file)"></font-awesome-icon>
                <font-awesome-icon :icon="closeIcon" @click="selectForDeletion(file)" v-if="disabled === 0"></font-awesome-icon>
            </div>
            <div class="file-name">
                {{ file.original_filename }}
            </div>
        </div>
        <modal
                v-if="showFileDeleteModal"
                :isConfirm="true"
                confirmText="Confirm Delete"
                modalWidth="300px"
                @close="showFileDeleteModal = false"
                @confirmed="deleteSelectedFile">

            <span slot="header">Delete File</span>
            <div slot="body" id="measure-delete-dialog-text">Are you sure you want to delete this file?</div>
        </modal>
    </div>
</template>
<script>
    import axios from 'axios';
    import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
    import { faFile, faDownload, faTimes } from '@fortawesome/free-solid-svg-icons'
    export default {

        inject: {
            apiClient: {
                default() {
                    return axios;
                }
            }
        },

        props: ['files', 'disabled'],

        components: {
            FontAwesomeIcon
        },

        data() {
            return {
                showFileDeleteModal: false,
                deleteFile: {},
                fileIcon: faFile,
                downloadIcon: faDownload,
                closeIcon: faTimes
            }
        },

        methods: {
            downloadFile(file) {
                window.open('/files/' + file.id + '/download');
            },
            previewIcon(file) {
                if(file.thumbnail && file.thumbnail !== null) {
                    return '/files/' + file.id + '/thumbnail'
                }

                return false;
            },
            selectForDeletion(file) {
                this.deleteFile = file;
                this.showFileDeleteModal = true;
            },
            deleteSelectedFile() {

                if(this.disabled === 1) {
                    return;
                }

                this.showFileDeleteModal = false;
                this.apiClient.delete('/api/files/' + this.deleteFile.id).then( response => {
                    this.$emit('deletedFile', this.deleteFile);
                    this.deleteFile = {};
                    window.notify.message('Successfully deleted file', 'success');
                }).catch( error => {
                    window.notify.apiError(error);
                });
            }
        }
    }
</script>
