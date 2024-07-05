<template>
  <v-container>
    <nuxt-content :document="services"/>
  </v-container>
</template>

<script>
export default {
  async asyncData({ $content }){
    const services = await $content('documentation', 'terms-of-service').fetch();
    return { services }
  },
}
</script>