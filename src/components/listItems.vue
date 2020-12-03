<template>
    <ul class="listItems" v-if="Array.isArray(items) && items.length > 0">
        <template v-for="(item, index) of items">
            <li class="listItems__item" v-if="item" :key="index">
                {{ item }}
            </li>
        </template>
    </ul>
</template>

<script>
    export default {
        name: 'listItems',

        props: {
            items: {
                type: Array,
                required: true,
            },
        },
    };
</script>

<style lang="scss">
    .listItems {
        padding: 15px 20px;
        margin: 0;

        background: #f8f8f8;
        border-radius: 2px;

        &__item {
            margin: 0 0 8px 15px;

            &:last-child {
                margin-bottom: 0;
            }
        }
    }
</style>
