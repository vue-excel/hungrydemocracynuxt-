<template>
  <v-container fluid :style="{ width: '95%', margin: '0 auto' }">
    <v-row
      v-if="$vuetify.breakpoint.mdAndUp"
      justify="center"
      class="recent-header"
    >
      <v-col cols="12" md="8">
        <RecipeFacetsGroup />
      </v-col>
    </v-row>
    <!-- <div class="text-h5 font-weight-bold pb-4 pl-2">Matched Recipes</div> -->
    <v-row>
      <v-col
        v-for="(recipe, index) in this.getSearchResults"
        cols="12"
        sm="6"
        md="3"
        :key="index"
        class="pl-5 pr-5"
      >
        <RecipeCard :recipe="recipe" />
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import RecipeCard from "@/components/RecipeCard.vue";
import RecipeFacetsGroup from "@/components/RecipeFacetsGroup";

export default {
  components: {
    RecipeCard,
    RecipeFacetsGroup,
  },
  methods: {},
  computed: {
    getSearchResults: function () {
      return this.$store.getters['searchStore/getSearchResults']
    }
  },
};
</script>

<style scoped>
.recent-header {
  margin-top: 0px !important;
}
</style>
