<template>
  <AddARecipe
    v-if="conversionCompleted"
    :recipeProp="recipeProp"
    :isDraft="isDraft"
  />
  <!-- :user="user" -->
</template>

<script>
import AddARecipe from "../../_userName/new.vue";
import { createNamespacedHelpers } from "vuex";
// const { mapGetters } = createNamespacedHelpers("userStore");

const { mapActions } = createNamespacedHelpers("addRecipeStore");

export default {
  data() {
    return {
      conversionCompleted: false,
    };
  },
  components: {
    AddARecipe,
  },
  props: ["recipeProp", "isDraft"],
  methods: {
    ...mapActions(["convertRecipeToState"]),
  },
  mounted() {
    this.convertRecipeToState(this.recipeProp);
    this.conversionCompleted = true;
  },
};
</script>

<style></style>
