<template>
    <RouterLink to="/">
        <div
        class="bg-dark py-5"
        :style="{
            background: `url(${notfound})`,
            aspectRatio: 2,
            backgroundRepeat: 'no-repeat',
            backgroundSize: '100%',
            backgroundPosition: '50% 0%'
        }"
        >
        </div>
    </RouterLink>
    
</template>

<script>
    import notfound from '@/assets/images/notfound.png'

    export default {
        name: 'NotFound',
        data() {
            return {
                notfound
            }
        }
    }
</script>