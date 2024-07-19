<template>
  <v-container>
    <v-select
      :items="items"
      v-model="selection"
      item-text="text"
      item-value="id"
      outlined
      dense
      :label="name"
      multiple
      @change="chooseFacet"
    >
    </v-select>
  </v-container>
</template>

<script>

export default {
  // props: {
  //   name: String,
  //   values: Object,
  //   type: String,
  //   currSelection: Array,
  // },
  data() {
    return {
      items: this.getValues(),
      selection: this.$props.currSelection,
    };
  },
  computed: {
    facets() {
      return this.$store.getters['searchStore/getFacets']
    },
    searchTerm() {
      return this.$store.getters['searchStore/getSearchTerm']
    },
  },
  methods: {
    getValues() {
      let facetValues = [];
      Object.keys(this.$props.values).forEach((e) => {
        facetValues.push({
          text: `${e} (${this.$props.values[e]})`,
          id: `${e}`,
        });
      });
      return facetValues;
    },
    async chooseFacet() {
.      const type = this.$props.type;
      // this.$store.dispatch('searchStore/updateFacets', { type, value }, { root: true });
      // const val = this.searchTerm;
      // const facets = this.facets;
      // this.$store.dispatch('searchStore/performSearch', { val, facets }, { root: true });
    },
  },
};
</script>

<style></style>
