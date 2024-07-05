<template>
  <v-row class='py-0'>
    <v-col v-if='this.getCuisinesFacetsResults' cols='12' md='3' class='py-0'>
      <Facets
        name='Cuisine'
        type='cuisine'
        :values='this.getCuisinesFacetsResults'
        :currSelection='this.getCuisineFacets'
      />
    </v-col>
    <v-col v-if='this.getCategoriesFacetsResults' cols='12' md='3' class='py-0'>
      <Facets
        name='Category'
        type='category'
        :values='this.getCategoriesFacetsResults'
        :currSelection='this.getCategoryFacets'
      />
    </v-col>
    <v-col v-if='this.getToolsFacetsResults' cols='12' md='3' class='py-0'>
      <Facets
        name='Tools'
        type='tool'
        :values='this.getToolsFacetsResults'
        :currSelection='this.getToolsFacets'
      />
    </v-col>
    <v-col v-if='this.getIngredientsFacetsResults' cols='12' md='3' class='py-0'>
      <Facets
        name='Ingredients'
        type='ingredient'
        :values='this.getIngredientsFacetsResults'
        :currSelection='this.getIngredientFacets'
      />
    </v-col>
  </v-row>
</template>

<script>
import Facets from '@/components/Facets.vue';

export default {
  name: 'recipe-facet-group',
  components: {
    Facets
  },
  computed: {
    getToolsFacetsResults() {
      return this.$store.getters['searchStore/getToolsFacetsResults']
    },
    getCategoriesFacetsResults() {
      return this.$store.getters['searchStore/getCategoriesFacetsResults']
    },
    getCuisinesFacetsResults() {
      return this.$store.getters['searchStore/getCuisinesFacetsResults']
    },
    getIngredientsFacetsResults() {
      return this.$store.getters['searchStore/getIngredientsFacetsResults']
    },
    getCuisineFacets() {
      return this.$store.getters['searchStore/getCuisineFacets']
    },
    getCategoryFacets() {
      return this.$store.getters['searchStore/getCategoryFacets']
    },
    getToolsFacets() {
      return this.$store.getters['searchStore/getToolsFacets']
    },
    getIngredientFacets() {
      return this.$store.getters['searchStore/getIngredientFacets']
    },
  }
};
</script>
