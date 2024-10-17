<script setup>
    import { defineProps } from 'vue'

    import MediaButton from './MediaButton'

    const props = defineProps({
        onRemove: Function,
        columns: {
            type: Array,
            required: true
        },
        dataKeys: {
            type: Array,
            required: true
        },
        paginatedEntries: {
            type: Array,
            required: true
        },
        formatDate: Function,
    })
</script>

<template>
    <div class="main__data__wrapper">
        <table class="table">
          <thead>
            <tr>
                <th v-for="(fieldName, index) in props.columns" :key="index">{{ fieldName }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="entry in JSON.parse(JSON.stringify(props.paginatedEntries))" :key="entry.id">
              <td v-for="(key, index) in props.dataKeys" :key="key">
                <template v-if="index < props.columns.length - 1">
                    <template v-if="key == 'registration_date'">
                        {{ formatDate(new Date(entry[key])) }}
                    </template>
                    <template v-else>
                        {{ entry[key] }}
                    </template>
                </template>
                <template v-else>
                    <div class="cellWithButtonWrapper">
                        <span>{{ entry[key] }}</span>
                        <media-button iconName="cancel.svg" iconAlt="Удалить" tableBtn @click="props.onRemove(entry)" />
                    </div>
                </template>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
</template>

<style scoped>
    .table {
        border-collapse: collapse;
        width: 100%;
        text-align: left;
        font-weight: 500;
        table-layout: fixed;
    }
    .table tr {
        border-bottom: 1px solid #F7F7F7;
    }
    .table th {
        font-size: 10px;
        line-height: 14px;
        color: #9EAAB4;
        font-weight: 500;
    }
    .table td {
        font-size: 12px;
        font-weight: 500;
        line-height: 16px;
        color: #ACACAC;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
    .table td:first-child {
        font-weight: 700;
        color: #0CB4F1;
    }
    .table th, .table td {
        padding: 16px 0 20px;
    }
    .table tbody > tr:last-child {
        border-bottom: none;
    }
    .main__data__wrapper {
        margin-top: 15px;
        padding: 0 16px;
        border-radius: 7px;
        
        background-color: #FFFFFF;
    }
    .cellWithButtonWrapper {
        display: grid;
        column-gap: 4px;
        grid-template-columns: 1fr auto;
        width: inherit
    }
    .cellWithButtonWrapper > span {
        text-overflow: ellipsis;
        white-space: nowrap;
        overflow: hidden;
    }
</style>