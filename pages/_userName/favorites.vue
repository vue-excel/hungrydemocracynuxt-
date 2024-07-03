<template>
  <v-container fluid :style="{ width: '95%', margin: '0 auto' }">
    <div class="top-background hidden-xs-only" :style="{ height: '75px' }">
      <v-img
        height="75px"
        eager
        alt="hungrydemocracy top bar background image"
        src="@/assets/top_bar_bg.png"
      >
      </v-img>
    </div>
    <div class="d-flex align-center justify-start pt-2">
      <v-btn icon color="black" @click="$router.go(-1)">
        <v-icon large>mdi-chevron-left</v-icon>
      </v-btn>
      <div class="ml-2 text-h5 font-weight-bold">My Favorite Recipes</div>
    </div>
    <SkeletonLoader v-if="isLoading" />
    <v-row v-else>
      <v-col
        cols="12"
        sm="4"
        md="3"
        v-for="(recipe, index) in favoriteRecipes"
        :key="index"
        class="pl-5 pr-5"
      >
        <RecipeCard :recipe="recipe" @loadFavorites="loadFavorites" />
      </v-col>
      <v-col v-if="favoriteRecipes.length < 1" cols="12">
        <div
          class="text-center d-flex align-center justify-center text-h6"
          style="height: 300px"
        >
          You have no Favorites.
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { getUserFavorites } from "../../service/db.service";
import SkeletonLoader from "@/components/loaders/SkeletonLoader";
import RecipeCard from "@/components/RecipeCard.vue";
export default {
  data() {
    return {
      isLoading: false,
      favoriteRecipes: [],
    };
  },
  components: {
    SkeletonLoader,
    RecipeCard,
  },
  async fetch() {
    await this.loadFavoriteRecipes();
  },
  computed: {
    uid: function () {
      return this.$store.getters["userStore/getUid"];
    },
  },
  methods: {
    loadFavorites() {
      this.loadFavoriteRecipes();
    },
    async loadFavoriteRecipes() {
      this.isLoading = true;
      this.favoriteRecipes = await getUserFavorites(this.uid);
      this.isLoading = false;
    },
  },
};
</script>

<style>
</style>