<template>
    <v-container>
      <nuxt-content :document="privacyPolicy"/>
    </v-container>
</template>
  
<script>
  export default {
    async asyncData({ $content }){
      const privacyPolicy = await $content('documentation', 'privacy-policy').fetch();
      return { privacyPolicy }
    },
  }
</script>