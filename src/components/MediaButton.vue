<script setup>
    import { defineProps } from 'vue'
    const props = defineProps({
        iconName: String,
        iconAlt: String,
        text: String,
        mt: String,
        size: String,
        shade: String,
        tableBtn: Boolean,
        clickable: {
            type: Function,
            default: () => true
        }
    })
</script>

<template>
    <button
        class="btn"
        :class="{
            'mtlarge': props.mt == 'large',
            'tablebtn': props.tableBtn,
            'smallgap': props.iconName && props.text,
            'disabled': props.clickable() == false,
            'large': props.size == 'large',
            'primary': props.shade == 'primary'
        }"
        :disabled="props.clickable() == false"
    >
        <img
            v-if="props.iconName"
            :src="require('../assets/' + props.iconName)"
            :alt="props.iconAlt"
            :class="props.size"
        />
        <span v-if="props.text">{{ props.text }}</span>
    </button>
</template>

<style>
    .btn {
        color: #4F4F4F;
        display: grid;
        grid-template-columns: auto 1fr;
        padding: 0;
        font-size: 12px;
        font-weight: 500;
        line-height: 16px;
        outline: none;
        background-color: transparent;
        border: none;
    }
    .smallgap {
        column-gap: 4px;
    }
    .btn:hover {
        cursor: pointer;
    }
    .mtlarge {        
        margin-top: 24px;
    }
    .tablebtn {
        float: right;
        margin-right: 16px;
    }
    .small {
        height: 16px !important;
        width: 16px;
    }
    .large {
        background: #E0E0E0;
        padding: 6px 36px;
        border-radius: 3px;
        font-size: 12px;
        font-weight: 500;
        line-height: 14.52px;
        color: #828282;
    }
    .primary {
        background-color: #0CB4F1;
        color: #FFFFFF;
    }
    .disabled {
        opacity: 0.5;
    }
    .disabled:hover {
        cursor: not-allowed;
    }
</style>