<script setup lang="ts">
    import Orders from '~/components/my/Orders.vue'

    const route = useRoute()
    const id: any = route.params.id
</script>

<template>
    <Orders :orderId = id />

</template>