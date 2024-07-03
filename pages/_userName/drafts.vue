<template>
  <v-container fluid :style="{ width: '95%', margin: '0 auto' }">
    <div class="top-background hidden-xs-only" :style="{ height: '75px' }">
      <v-img height="75px" eager alt="hungrydemocracy top bar background image" src="@/assets/top_bar_bg.png"> </v-img>
    </div>
    <div class="d-flex align-center justify-start pt-2">
      <v-btn icon color="black" @click="$router.go(-1)">
        <v-icon large>mdi-chevron-left</v-icon>
      </v-btn>
      <div class="ml-2 text-h5 font-weight-bold">My Draft Recipes</div>
    </div>
    <SkeletonLoader v-if="this.getLoadingState" />
    <v-row v-else>
      <v-col
        cols="12"
        sm="4"
        md="3"
        v-for="(recipe, index) in this.getDraftRecipes"
        :key="index"
        class="pl-5 pr-5"
      >
        <RecipeCard :recipe="recipe" :isDraft="true" />
      </v-col>
      <v-col v-if="this.getDraftRecipes.length < 1" cols="12">
        <div
          class="text-center d-flex align-center justify-center text-h6"
          style="height: 300px"
        >
          You have no drafts.
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import RecipeCard from "@/components/RecipeCard.vue";
import SkeletonLoader from "@/components/loaders/SkeletonLoader";
import { createNamespacedHelpers } from "vuex";

const { mapGetters, mapActions } = createNamespacedHelpers("dbStore");

export default {
  components: {
    RecipeCard,
    SkeletonLoader,
  },
  methods: {
    ...mapActions(["loadDraftRecipes"]),
  },
  computed: {
    ...mapGetters(["getDraftRecipes", "getLoadingState"]),
  },
  async created() {
    await this.loadDraftRecipes();
  },
};
</script>

<style></style>
