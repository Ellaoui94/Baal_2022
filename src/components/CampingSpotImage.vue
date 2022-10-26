<script setup lang="ts">

interface Props {
  imageId:number
}

const props = defineProps<Props>()

const userAccessToken = localStorage.getItem('auth_token')

</script>

<template>
  <img :src="`https://7qp4jl4l.directus.app/assets/${imageId}?access_token=${userAccessToken}`"/>
</template>