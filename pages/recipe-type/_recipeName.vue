<template>
  <v-container fluid style="width: 95%; margin: 0 auto" v-if="!isLoading">
    <div class="top-background hidden-xs-only">
      <!-- <v-img height="140px" eager alt="hungrydemocracy top bar background image" :src="require('~/static/top_bar_bg.png')">
      </v-img> -->
    </div>
    <div style="position: relative" class="text-center mt-16" v-if="recipeName">
      <v-row>
        <v-breadcrumbs
          :items="breadCrumbs"
          class="mt-3 no-color horizontal-scroll"
        >
          <template v-slot:divider>
            <v-icon>mdi-chevron-right</v-icon>
          </template>
          <template v-slot:item="{ item }">
            <v-breadcrumbs-item :href="item.href" :disabled="item?.disabled">
              {{ textCapitalize(item.text) }}
            </v-breadcrumbs-item>
          </template>
        </v-breadcrumbs>
      </v-row>
      <h1 class="text-h5 font-weight-bold">
        {{ recipeName }}
      </h1>
    </div>
    <div class="d-flex flex-wrap justify-center">
      <v-col
        v-if="siblings && siblings[0]"
        class="d-flex col-md-4"
        cols="12"
        sm="6"
      >
        <v-select
          :items="siblings"
          v-model="selectedRecipe"
          :label="
            breadCrumbs[breadCrumbs.length - 2]?.text === 'Home'
              ? `Explore other recipe types`
              : `Explore other ${
                  breadCrumbs[breadCrumbs.length - 2]?.text
                } types`
          "
          outlined
          dense
          :menu-props="{ bottom: true, offsetY: true }"
        >
          <template v-slot:item="{ item }">
            <v-list-item>
              <nuxt-link :to="{ path: `/recipe-type/${urlRewrite(item)}` }">
                <v-list-item-title>
                  {{ item }}
                </v-list-item-title>
              </nuxt-link>
            </v-list-item>
          </template>
        </v-select>
      </v-col>
      <v-col
        v-if="childrens && childrens?.[0]"
        class="d-flex col-md-4"
        cols="12"
        sm="6"
      >
        <v-select
          :items="childrens"
          v-model="selectedRecipe"
          :label="`Choose ${recipeName} type`"
          outlined
          dense
          :menu-props="{ bottom: true, offsetY: true }"
        >
          <template v-slot:item="{ item }">
            <v-list-item>
              <nuxt-link :to="{ path: `/recipe-type/${urlRewrite(item)}` }">
                <v-list-item-title>
                  {{ item }}
                </v-list-item-title>
              </nuxt-link>
            </v-list-item>
          </template>
        </v-select>
      </v-col>
    </div>
    <v-row class="mt-2" justify="center">
      <v-card min-width="300px" elevation="0">
        <v-chip-group
          active-class="primary--text"
          class="recipeTypesTags"
          @change="getAllRecipeByMainIngredient"
          v-model="selectedMainIngredient"
        >
          <v-chip
            class="ma-1 chip-shadow"
            color="white"
            v-for="(val, i) in mainIngredientFacets"
            :key="i"
            >{{ val?.name }}</v-chip
          >
        </v-chip-group>
      </v-card>
    </v-row>
    <v-row class="pt-5">
      <v-col
        cols="12"
        sm="4"
        md="3"
        v-for="recipe in recipes"
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
import {
  getRecipeTypesDetails,
  getIngredientsName,
} from "@/service/db.service";
import { getFacets, getRecipes } from "../../service/search.service";
import RecipeCard from "@/components/RecipeCard.vue";

export default {
  name: "userProfile",
  components: {
    RecipeCard,
  },
  head() {
    return {
      title: `${this.recipeName}`,
      meta: [
        { charset: "utf-8" },
        {
          hid: "og:title",
          property: "og:title",
          content: `All Recipes of ${this.recipeName}`,
        },
        {
          hid: "og:url",
          property: "og:url",
          content: `${process.env.NUXT_ENV_BASE_URL}${this.$nuxt.$route.path}`,
        },
        {
          hid: "og:description",
          property: "og:description",
          content: `Find all recipes of ${this.recipeName} in detail at hungrydemocracy with ingredient, tools, prepration time, cooking time recommendations ...`,
        },
        // {
        //   hid: "og:image",
        //   property: "og:image",
        //   content: this.recipe?.imageUrls[0],
        // },
        {
          hid: "description",
          name: "description",
          content: `Find all recipes of ${this.recipeName} in detail at hungrydemocracy with ingredient, tools, prepration time, cooking time recommendations ...`,
        },
        { hid: "og-type", property: "og:type", content: "website" },
        { name: "viewport", content: "width=device-width, initial-scale=1" },
      ],
      link: [
        {
          hid: "canonical",
          rel: "canonical",
          href: this.$nuxt.$route.query.ingredient ? `${process.env.NUXT_ENV_BASE_URL}${this.$nuxt.$route.path}?ingredient=${this.$nuxt.$route.query.ingredient}` : this.$nuxt.$route.query.page ? `${process.env.NUXT_ENV_BASE_URL}${this.$nuxt.$route.path}?page=${this.$nuxt.$route.query.page}` : `${process.env.NUXT_ENV_BASE_URL}${this.$nuxt.$route.path}`,
        },
      ],
    };
  },
  data() {
    return {
      isLoading: false,
      siblings: [],
      childrens: [],
      selectedRecipe: null,
      breadCrumbs: [],
      recipes: [],
      recipeName: "",
      selectedMainIngredient: null,
      mainIngredientFacets: [],
      page: 1,
      totalPages: 0,
      customFilter: null
    };
  },
  mounted() {
    if (this.$route.path.includes("%20")) {
      this.$router.push({
        name: "pageNotFound",
      });
    }
  },
  methods: {
    urlRewrite(val) {
      let result = val?.split("/").join("&");
      result = result?.split(" ").join("-");
      return result;
    },
    textCapitalize(str) {
      let result = str?.trim().split(" ");
      result = result?.map((el) =>
        el.length ? el[0]?.toUpperCase() + el.trim()?.slice(1) : ""
      );
      result = result?.join(" ");
      return result;
    },
    async getAllRecipeByMainIngredient() {
      this.page = 1;
      if (this.selectedMainIngredient !== undefined) {
        this.$router.push({
          path: `/recipe-type/${this.urlRewrite(this.recipeName)}`,
          query: { ingredient: this.mainIngredientFacets[this.selectedMainIngredient]?.name.split(" ").join("-") },
        });
      } else {
        this.$router.push({
          path: `/recipe-type/${this.urlRewrite(this.recipeName)}`
        });
      }
    },
    paginate(){
      this.$router.push({
        path: `/recipe-type/${this.urlRewrite(this.recipeName)}`,
        query: {page: this.page}
      })
    }
  },
  async fetch() {
    this.isLoading = true;
    this.recipeName = this.$route.params.recipeName.split("&").join("/");
    this.recipeName = this.recipeName.split("-").join(" ");
    const recipeName = this.textCapitalize(this.recipeName);
    const recipeTypesData = await getRecipeTypesDetails(recipeName);
    this.siblings = recipeTypesData?.siblings;
    this.childrens = recipeTypesData?.children;
    this.breadCrumbs = [];
    if (recipeTypesData?.pathsToRoot) {
      recipeTypesData.pathsToRoot.forEach((el) => {
        this.breadCrumbs?.push({
          text: el === "All Recipes" ? "Home" : el,
          disabled: false,
          href: el === "All Recipes" ? "/" : `/recipe-type/${this.urlRewrite(el.trim())}`,
        });
      });
      this.breadCrumbs?.push({
        text: recipeTypesData?.allNames[0],
        disabled: true,
        href: `/recipe-type/${this.urlRewrite(recipeTypesData?.allNames[0].trim())}`,
      });
    }
    this.page = this.$route.query.page ? parseInt(this.$route.query.page) : 1;

    if(this.childrens[0]!== ""){
      this.customFilter = `type:recipes AND recipeTypes:`
      for (let index = 0; index < this.childrens.length-1; index++) {
        this.customFilter = this.customFilter.concat(`"${this.childrens[index]}"`, ' OR recipeTypes:');
      }
      this.customFilter = this.customFilter.concat(`"${this.childrens[this.childrens.length-1]}"`, ` OR recipeTypes:"${this.recipeName}"`)
    }

    const result = await getFacets("mainIngredients", "recipeTypePage", this.recipeName, null, null, this.customFilter);
    const mainIngredientFacets = Object.keys(result.facets.mainIngredients);
    for(const facet of mainIngredientFacets){
      const facetName = await getIngredientsName(facet)
      if(facetName){
        const ingredient = { id: facet, name: facetName };
        this.mainIngredientFacets.push(JSON.parse(JSON.stringify(ingredient)));
      }
    }
    let results = null;
    if (this.$route.query.ingredient) {
      this.selectedMainIngredient = this.mainIngredientFacets?.findIndex(
        (i) => i.name === this.$route.query.ingredient.split("-").join(" ")
      );
      results = await getRecipes(
        "mainIngredients",
        "recipeTypePage",
        this.page - 1,
        this.recipeName,
        this.mainIngredientFacets[this.selectedMainIngredient]?.id,
        null,
        this.customFilter
      );
    } else {
      results = await getRecipes(
        "mainIngredients",
        "recipeTypePage",
        this.page - 1,
        this.recipeName,
        null,
        null,
        this.customFilter
      );
    }
    this.recipes = results?.hits;
    this.recipes = this.recipes?.map((el) => {
      el.imageUrls = el?.imageUrls?.map((el) => {
        return el?.split("&")[0];
      });
      return el;
    });
    this.totalPages = parseInt(results?.nbPages);
    this.isLoading = false;
  },
  watch: {
    "$route.query.ingredient": {
      handler: async function (search) {
        let result = null;
        if (search) {
          this.selectedMainIngredient = this.mainIngredientFacets.findIndex(
            (i) => i.name === search.split("-").join(" ")
          );
          result = await getRecipes(
            "mainIngredients",
            "recipeTypePage",
            this.page - 1,
            this.recipeName,
            this.mainIngredientFacets[this.selectedMainIngredient].id,
            null,
            this.customFilter
          );
        } else {
          this.selectedMainIngredient = undefined;
          result = await getRecipes(
            "mainIngredients",
            "recipeTypePage",
            this.page - 1,
            this.recipeName,
            null,
            null,
            this.customFilter
          );
        }
        this.recipes = result.hits;
        this.totalPages = parseInt(result.nbPages);
      },
    },
    "$route.query.page": {
      handler: async function (page) {
        let result = null;
        if (page) {
          this.page = parseInt(page);
          this.selectedMainIngredient = undefined;
          result = await getRecipes(
            "mainIngredients",
            "recipeTypePage",
            this.page - 1,
            this.recipeName,
            null,
            null,
            this.customFilter
          );
        } else {
          this.page = 1;
          this.selectedMainIngredient = undefined;
          result = await getRecipes(
            "mainIngredients",
            "recipeTypePage",
            this.page - 1,
            this.recipeName,
            null,
            null,
            this.customFilter
          );
        }
        this.recipes = result.hits;
        this.totalPages = parseInt(result.nbPages);
      },
    },
  },
};
</script>
