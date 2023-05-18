<script setup>
    import { computed } from "vue";

    const props = defineProps({
        currentPage: Number,
        itemCount: Number,
        pageLimit: Number
    });
    const emit = defineEmits(['changePage']);

    const pageCount =  computed(() => Math.ceil(props.itemCount / props.pageLimit));
    const pageList = computed(() => Array.from(Array(pageCount.value), (_, index) => index+1));
    const previousPage = computed(() => Math.max(props.currentPage - 1, 1));
    const nextPage = computed(() => Math.min(props.currentPage + 1, pageCount.value));

    function changePagination(iPage) {
        emit('changePage', iPage);
    }
</script>

<template>
    <ul class="pagination">
        <li class="page-item"><a class="page-link" href="#" @click="changePagination(previousPage)">Previous</a></li>
        <li v-for="pageItem in pageList" :key="pageItem" class="page-item" :class="{ active: props.currentPage === pageItem }" @click="changePagination(pageItem)">
            <a href="#" class="page-link">{{ pageItem }}</a>
        </li>
        <li class="page-item"><a class="page-link" href="#" @click="changePagination(nextPage)">Next</a></li>
    </ul>
</template>