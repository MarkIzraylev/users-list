<script setup>
    import { defineProps } from 'vue'

    import MediaButton from './MediaButton'

    defineProps({
        actionOnYes: Function,
        closeModal: Function
    })
</script>

<template>
    <div class="overlay" ref="overlay"></div>
    <dialog class="modal" id="removeUserModal" open>
        <p><slot /></p>
        <div class="modalActions">
            <media-button text="Да" mt="medium" @click="actionOnYes()" size="large" shade="secondary" />
            <media-button text="Нет" mt="medium" @click="closeModal()" size="large" shade="primary" />
        </div>
    </dialog>
</template>

<style scoped>
    .overlay {
        position: fixed;
        width: 100%;
        height: 100%;
        z-index: 1;
        display: block;
        backdrop-filter: blur(3px);
        background-color: rgba(0, 0, 0, 0.2);
    }
    .modal {
        position: fixed;
        margin: 0;
        z-index: 2;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        border-radius: 8px;
        background-color: #FFFFFF;
        border: none;
        outline: none;
        padding: 36px 36px 32px 36px;
    }
    .modal > p {
        margin: 0;
        font-size: 12px;
        font-weight: 400;
        line-height: 14.52px;
    }
    .modalActions {
        display: flex;
        justify-content: center;
        flex-direction: row;
        gap: 36px;
        margin-top: 24px;
    }
</style>