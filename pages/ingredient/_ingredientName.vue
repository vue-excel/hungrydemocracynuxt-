<template>
  <v-container fluid style="width: 95%; margin: 0 auto" v-if="!isLoading">
    <div class="top-background hidden-xs-only">
      <!-- <v-img height="140px" eager alt="hungrydemocracy top bar background image" :src="require('~/static/top_bar_bg.png')">
      </v-img> -->
    </div>
    <div
      style="position: relative"
      class="text-center mt-16"
      v-if="ingredientName"
    >
      <h1 class="text-h5 font-weight-bold">
        {{ ingredientName }}
      </h1>
    </div>
    <v-row class="mt-2" justify="center">
      <v-card min-width="300px" elevation="0">
        <v-chip-group
          active-class="primary--text"
          @change="getAllRecipeByRecipeTypes"
          v-model="selectedRecipeType"
        >
          <v-chip
            class="ma-1 chip-shadow"
            color="white"
            v-for="(val, i) in recipeTypeFacets"
            :key="i"
            >{{ val }}</v-chip
          >
        </v-chip-group>
      </v-card>
    </v-row>
    <v-row class="pt-5">
      <v-col
        cols="12"
        sm="4"
        md="3"
        v-for="recipe in allRecipes"
        :key="recipe.objectID"
        class="pl-5 pr-5"
      >
        <RecipeCard :recipe="recipe" />
      </v-col>
    </v-row>
    <v-row v-if="totalPages > 1">
      <v-col>
        <div class="text-center">
        <v-pagination v-model="page" :length="totalPages" @input="paginate"></v-pagination>
        </div>
      </v-col>
    </v-row>
  </v-container>
  <v-container
    v-else
    fluid
    class="align-center d-flex justify-center fill-height"
    :style="{ height: '90vh' }"
  >
    <v-progress-circular size="50" color="primary" indeterminate />
  </v-container>
</template>

<script>
import RecipeCard from "~/components/RecipeCard.vue";
import { isMainIngredient } from "../../service/db.service";
import { getFacets, getRecipes } from "../../service/search.service";
export default {
  data() {
    return {
      allRecipes: null,
      recipesPerPage: null,
      ingredientName: "",
      isLoading: true,
      page: 1,
      totalPages: 0,
      recipeTypeFacets: [],
      ingredientData: null,
      selectedRecipeType: null,
    };
  },
  components: { RecipeCard },
  async fetch() {
    this.isLoading = true;
    this.ingredientName = this.$route.params.ingredientName.split("-").join(" ");
    this.page = this.$route.query.page ? parseInt(this.$route.query.page) : 1;
    this.ingredientData = await isMainIngredient(this.ingredientName);
    if (this.ingredientData) {
      const result = await getFacets(
        "recipeTypes",
        "ingredientPage",
        null,
        this.ingredientData.id
      );
      this.recipeTypeFacets = Object.keys(result.facets.recipeTypes);
      let results = null;
      if (this.$route.query.type) {
        this.selectedRecipeType = this.recipeTypeFacets.indexOf(
          this.$route.query.type.split("-").join(" ")
        );
        results = await getRecipes(
          "recipeTypes",
          "ingredientPage",
          this.page - 1,
          this.$route.query.type.split("-").join(" "),
          this.ingredientData.id
        );
      } else {
        results = await getRecipes(
          "recipeTypes",
          "ingredientPage",
          this.page - 1,
          null,
          this.ingredientData.id
        );
      }
      this.allRecipes = results.hits;
      this.totalPages = parseInt(results.nbPages);
    } else {
      this.$router.push({ path: "/pageNotFound" });
    }
    this.isLoading = false;
  },
  methods: {
    async getAllRecipeByRecipeTypes() {
      this.page = 1;
      if (this.selectedRecipeType !== undefined) {
        this.$router.push({
          path: `/ingredient/${this.ingredientName.split(" ").join("-")}`,
          query: { type: this.recipeTypeFacets[this.selectedRecipeType].split(" ").join("-") },
        });
      } else {
        this.$router.push({
          path: `/ingredient/${this.ingredientName.split(" ").join("-")}`
        });
      }
    },
    paginate(){
      this.$router.push({
        path: `/ingredient/${this.ingredientName.split(" ").join("-")}`,
        query: {page: this.page}
      })
    }
  },
  watch: {
    "$route.query.type": {
      handler: async function (search) {
        let result = null
        if (search) {
          this.selectedRecipeType = this.recipeTypeFacets.indexOf(
            search.split("-").join(" ")
          );
          result = await getRecipes(
            "recipeTypes",
            "ingredientPage",
            this.page - 1,
            this.recipeTypeFacets[this.selectedRecipeType],
            this.ingredientData.id
          );
        } else {
          this.page = 1;
          result = await getRecipes(
            "recipeTypes",
            "ingredientPage",
            this.page - 1,
            null,
            this.ingredientData.id
          );
          this.selectedRecipeType = undefined;
        }
        this.allRecipes = result.hits;
        this.totalPages = parseInt(result.nbPages);
      },
    },
    "$route.query.page": {
      handler: async function (page) {
        let result = null
        if (page) {
          this.page = parseInt(page);
          result = await getRecipes(
            "recipeTypes",
            "ingredientPage",
            this.page - 1,
            null,
            this.ingredientData.id
          );
        } else {
          this.page = 1;
          result = await getRecipes(
            "recipeTypes",
            "ingredientPage",
            this.page - 1,
            null,
            this.ingredientData.id
          );
        }
        this.selectedRecipeType = undefined;
        this.allRecipes = result.hits;
        this.totalPages = parseInt(result.nbPages);
      },
    },
  },
};
</script>

<style scoped>

</style>