<template>
  <v-container
    v-if="this.recipe"
    fluid
    :style="{ width: '95%', margin: '0 auto' }"
  >
    <!-- <script v-html="jsonld" type="application/ld+json"></script> -->
    <div class="top-background hidden-xs-only" :style="{ height: '300px' }">
      <div :style="{ height: '300px' }"></div>
      <div :style="{ height: '300px', position: 'absolute', top: '0px' }">
        <!-- <v-img
          eager
          alt="hungrydemocracy top bar background image"
          src="/static/top_bar_bg.png?webp"
          :style="{ minWidth: '100vw' }"
        ></v-img> -->
      </div>
    </div>
    <div style="position: relative">
      <v-row class="pl-5">
        <v-btn icon @click="goBack" class="text--black">
          <v-icon large class="text--black">mdi-chevron-left</v-icon>
        </v-btn>
      </v-row>
      <v-row>
        <v-breadcrumbs
          :items="breadCrumbs"
          class="mt-3 no-color horizontal-scroll"
        >
          <template v-slot:divider>
            <v-icon>mdi-chevron-right</v-icon>
          </template>
          <template v-slot:item="{ item }">
            <v-breadcrumbs-item :href="item.href" :disabled="item.disabled">
              {{ textCapitalize(item.text) }}
            </v-breadcrumbs-item>
          </template>
        </v-breadcrumbs>
      </v-row>
      <v-row class="mt-1">
        <v-col sm="12" md="3" class="pl-5 pr-5" cols="12">
          <SwipeableImages :handleDialog="handleDialog" :recipe="this.recipe" />
        </v-col>
        <v-col cols="12" sm="12" md="9">
          <RecipeDetail
            :alreadyLiked="alreadyLiked"
            :editRecipe="this.editRecipe"
            :recipe="this.recipe"
            :likeLoading="likeLoading"
            :likeRecipe="this.likeRecipe"
            :uid="this.uid"
            :likesCount="+this.likes"
          />
          <v-row class="mt-4">
            <v-col cols="12">
              <AdditionalImageDetails :recipe="this.recipe" />
            </v-col>
          </v-row>
        </v-col>
      </v-row>
      <v-row class="mt-5">
        <v-col sm="4" md="3" cols="12">
          <RecipeIngredients :recipe="this.recipe" />
        </v-col>
        <v-col sm="8" md="9" :style="{ paddingBottom: '50px' }" cols="12">
          <div class="text-h6 font-weight-bold text-spl-underline">
            <a class="text--primary text-decoration-underline" @click="showAuthorsDirections = false" v-if="haveStructuredDirections">
              Cooking Instructions
            </a>
            <span v-if="haveStructuredDirections">/</span>
            <a
              class="text--primary text-decoration-underline"
              @click="showAuthorsDirections = true"
              >Authors Directions
            </a>
          </div>
          <CookingInstructions
            :recipe="this.recipe"
            v-if="showAuthorsDirections"
          />
          <StructuredDirections 
            :recipe="this.recipe"
            v-if="!showAuthorsDirections && haveStructuredDirections"
          />
        </v-col>
      </v-row>
      <RecipeRecommendation :recipeList="recommendedRecipes" />
      <ImageGallery
        v-if="dialog"
        :images="images"
        :dialog="dialog"
        :handleDialog="handleDialog"
      />
    </div>
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
  getRecipeFromUrlName,
  getAllRecommenedRecipes,
  getIngredientsName,
  getRecipeTypesDetails,
} from "@/service/db.service";
import { doesUserLikeRecipe, likeRecipe } from "../../../service/db.service";
import ImageGallery from "../../../components/recipedetail/ImageGallery.vue";
import SwipeableImages from "../../../components/recipedetail/SwipeableImageList";
import AdditionalImageDetails from "../../../components/recipedetail/AdditionalRecipeDetails";
import RecipeDetail from "../../../components/recipedetail/RecipeDetails";
import RecipeIngredients from "../../../components/recipedetail/IngredientDetails";
import CookingInstructions from "../../../components/recipedetail/CookingInstructions";
import RecipeRecommendation from "../../../components/recipedetail/RecipeRecommendation.vue";
import StructuredDirections from "../../../components/recipedetail/StructuredDirections.vue";

export default {
  head() {
    return {
      title: `${this.recipe?.title} recipe by @${this.recipe?.userName}`,
      meta: [
        { charset: "utf-8" },
        { hid: "og:title", property: "og:title", content: this.recipe?.title },
        {
          hid: "og:url",
          property: "og:url",
          content: `${process.env.NUXT_ENV_BASE_URL}${this.$nuxt.$route.path}`,
        },
        {
          hid: "og:description",
          property: "og:description",
          content:
            this.recipe && this.recipe.desc !== null
              ? this.recipe.desc
              : "Find cooking recipe in detail at hungrydemocracy with ingredient, tools, prepration time, cooking time recommendations ...",
        },
        {
          hid: "og:image",
          property: "og:image",
          content: this.recipe?.imageUrls[0],
        },
        {
          hid: "description",
          name: "description",
          content: `${
            this.recipe && this.recipe.desc !== null
              ? this.recipe.desc
              : "Find cooking recipe in detail at hungrydemocracy with ingredient, tools, prepration time, cooking time recommendations ..."
          }`,
        },
        { hid: "og-type", property: "og:type", content: "website" },
        { name: "viewport", content: "width=device-width, initial-scale=1" },
        { name: "thumbnail", content: this.recipe?.imageUrls[0] },
      ],
      link: [
        {
          hid: "canonical",
          rel: "canonical",
          href: `${process.env.NUXT_ENV_BASE_URL}${this.$nuxt.$route.path}`,
        },
      ],
      script: [{ type: "application/ld+json", json: this.jsonld }],
    };
  },
  props: {
    recipeProp: {
      type: Object,
    },
  },
  components: {
    ImageGallery,
    SwipeableImages,
    AdditionalImageDetails,
    RecipeDetail,
    RecipeIngredients,
    CookingInstructions,
    RecipeRecommendation,
    StructuredDirections,
  },
  data: function () {
    // mutating a prop directly is an anti pattern. That the reason a recipe was created which is assigned to recipeProp
    return {
      breadCrumbs: [],
      recommendedRecipes: [],
      recipe: this.recipeProp,
      images: [],
      alreadyLiked: false,
      likeLoading: true,
      likes: 0,
      jsonld: {},
      swiperOption: {
        pagination: {
          el: ".swiper-pagination",
        },
        navigation: {
          nextEl: ".swiper-button-next",
          prevEl: ".swiper-button-prev",
        },
      },
      dialog: false,
      showAuthorsDirections: false,
    };
  },
  mounted() {
    if (this.$route.path.includes("%20")) {
      this.$router.push({
        name: "pageNotFound",
      });
    }
  },
  async fetch() {
    if (this.recipeProp) {
      // this recipe was clicked from an existing recipe link and since router passes it as props, the data is already available
      this.recipe = this.recipeProp;
      this.likes = this.recipe.likesCount || 0;
    } else {
      // this was rendered from a url directly (say from bookmarks). This needs to be fetched from the db using the url
      try {
        this.recipe = await getRecipeFromUrlName(
          this.$route.params.recipeUrlName,
          this.$route.params.userName
        );
        if (!this.recipe) {
          this.$router.push({
            name: "pageNotFound",
          });
        }
      } catch (e) {
        this.$router.push({
          name: "pageNotFound",
        });
      }
    }

    // get breadcrumbs
    if (
      this.recipe &&
      this.recipe?.recipeTypes &&
      this.recipe?.recipeTypes.length
    ) {
      const recipeTypesData = await getRecipeTypesDetails(
        this.recipe?.recipeTypes[0]
      );
      this.breadCrumbs = [];
      if (recipeTypesData?.pathsToRoot) {
        recipeTypesData.pathsToRoot.forEach((el) => {
          this.breadCrumbs.push({
            text: el === "All Recipes" ? "Home" : el,
            disabled: false,
            href:
              el === "All Recipes"
                ? "/"
                : `/recipe-type/${this.urlRewrite(el.trim())}`,
          });
        });
        this.breadCrumbs.push({
          text: recipeTypesData?.allNames[0],
          disabled: false,
          href: `/recipe-type/${this.urlRewrite(
            recipeTypesData?.allNames[0].trim()
          )}`,
        });
        this.breadCrumbs.push({
          text: this.recipe.title,
          disabled: true,
          href: this.recipe.relativeUrl,
        });
      }
    }
    if (this.recipe?.imageUrls) {
      this.recipe.imageUrls = this.recipe?.imageUrls.map((imageUrl) => {
        return imageUrl?.split("&")[0];
      });
    }

    this.recipe?.imageUrls.forEach((imageUrl) =>
      this.images.push({
        title: this.recipe.title,
        description: `by ${this.recipe.userName}`,
        href: imageUrl,
      })
    );
    this.likeLoading = true;
    this.alreadyLiked = await doesUserLikeRecipe(this.uid, this.recipe.id);
    this.likeLoading = false;

    let catSimepleNames = "how to make ";
    this.recipe?.categories.forEach((category) => {
      category?.allNames.forEach((el) => {
        catSimepleNames = catSimepleNames.concat(el, ", ");
      });
    });
    catSimepleNames = catSimepleNames.concat(" recipe");
    const cuisinesSimpleNames = [];
    this.recipe?.cuisines.forEach((cuisine) =>
      cuisinesSimpleNames.push(cuisine?.allNames[0])
    );
    const simpleIngredientNames = [];
    this.recipe?.ingredientSections.forEach((section) =>
      section?.ingredients.forEach((ingredient) =>
        simpleIngredientNames.push(
          `${
            ingredient?.quantity +
            " " +
            ingredient?.unit +
            " " +
            ingredient?.choosenName
          }`
        )
      )
    );
    let recipeInstructions = []
    if(this.recipe?.directionInstructions) {
      recipeInstructions = this.recipe?.directionInstructions.map(
        (instruction) => ({
          "@type": "HowToSection",
          name: `${instruction.type}${
            instruction.title !== instruction.type ? `-${instruction.title}` : ""
          }`,
          itemListElement: instruction.options.map((opt) => ({
            "@type": "HowToStep",
            text: opt.additionalText,
          })),
        })
      );
    }
    else {
      this.showAuthorsDirections = true;
      recipeInstructions = this.recipe?.directions.map((direction) => ({
        "@type": "HowToStep",
        text: direction,
      }))
    }
    const recipeCookingTotalTime =
      parseInt(this.recipe?.prepTime) + parseInt(this.recipe?.cookingTime);
    let createdDate = new Date(0);
    createdDate.setUTCSeconds(this.recipe?.createdAt?.seconds);
    createdDate =
      createdDate.getFullYear() +
      "-" +
      (createdDate.getMonth() + 1) +
      "-" +
      createdDate.getDate();

    this.jsonld = {
      "@context": "https://schema.org/",
      "@type": "Recipe",
      name: this.recipe?.title,
      image: {
        "@type": "ImageObject",
        url: `${this.recipe?.imageUrls[0].split("&")[0]}`,
      },
      author: {
        "@type": "Person",
        name: this.recipe?.userName.split("-").join(" "),
      },
      datePublished: createdDate,
      description:
        this.recipe && this.recipe.desc !== null
          ? this.recipe.desc
          : "Find cooking recipe in detail at hungrydemocracy with ingredient, tools, prepration time, cooking time recommendations ...",
      prepTime: `PT${this.recipe?.prepTime}M`,
      cookTime: `PT${this.recipe?.cookingTime}M`,
      totalTime: `PT${recipeCookingTotalTime}M`,
      keywords: catSimepleNames,
      isFamilyFriendly: true,
      tool: this.recipe?.tools?.length
        ? this.recipe.tools.map((el) => {
            return { "@type": "HowToTool", name: el.allNames[0] };
          })
        : [],
      recipeYield: `${this.recipe?.serves}`,
      recipeCategory: catSimepleNames.length > 0 ? catSimepleNames[0] : "",
      recipeCuisine:
        cuisinesSimpleNames.length > 0 ? cuisinesSimpleNames[0] : "",
      recipeIngredient: simpleIngredientNames,
      recipeInstructions,
    };
    let recommendationArgs = [];
    if (this.recipe?.mainIngredients) {
      this.recipe.mainIngredients.forEach((el) => {
        recommendationArgs.push({
          type: "main-ingredient",
          data: el,
          currentRecipeId: this.recipe.id,
        });
      });
    }
    if (this.recipe?.recipeTypes) {
      this.recipe.recipeTypes.forEach((el) => {
        recommendationArgs.push({
          type: "recipe-type",
          data: el,
          currentRecipeId: this.recipe.id,
        });
      });
    }
    if (!recommendationArgs.length) {
      if (
        this.recipe?.categories &&
        !(this.recipe.categories?.[0].allNames[0] === undefined)
      ) {
        recommendationArgs.push({
          type: "categories",
          data: this.recipe.categories?.[0].allNames[0],
          currentRecipeId: this.recipe.id,
        });
      }
      if (
        this.recipe?.cuisines &&
        !(this.recipe.cuisines?.[0]?.allNames[0] === undefined)
      ) {
        recommendationArgs.push({
          type: "cuisines",
          data: this.recipe.cuisines?.[0]?.allNames[0],
          currentRecipeId: this.recipe.id,
        });
      }
    }
    this.recommendedRecipes = [];
    const result = [];
    for await (const el of recommendationArgs) {
      const recipes = await getAllRecommenedRecipes(el);
      if (recipes && recipes.length) {
        result.push({
          type: el.type,
          for:
            el.type === "main-ingredient"
              ? await getIngredientsName(el.data)
              : el.data,
          result: recipes,
        });
      }
    }
    this.$nextTick(() => {
      this.recommendedRecipes = result;
    });
  },
  computed: {
    uid: function () {
      return this.$store.getters["userStore/getUid"];
    },
    userName: function () {
      return this.$store.getters["userStore/getUserName"];
    },
    disableLikeButton: function () {
      return this.alreadyLiked || this.likeLoading;
    },
    haveStructuredDirections() {
      if(this.recipe?.directionInstructions) {
        return this.recipe.directionInstructions.length > 0
      }
      return false;
    }
  },
  methods: {
    textCapitalize(str) {
      let result = str.trim().split(" ");
      result = result?.map((el) =>
        el.length ? el[0]?.toUpperCase() + el.trim()?.slice(1) : ""
      );
      result = result?.join(" ");
      return result;
    },
    urlRewrite(str) {
      let result = str.split("/").join("&");
      result = result.split(" ").join("-");
      return result;
    },
    async likeRecipe(recipeId) {
      if (this.uid && !this.alreadyLiked) {
        this.likeLoading = true;
        await likeRecipe(this.uid, this.recipe.id || recipeId);
        this.likeLoading = false;
        this.alreadyLiked = true;
        this.likes = +this.likes + 1;
      } else {
      }
    },
    editRecipe() {
      this.$router.push({
        name: "userName-recipeUrlName-edit",
        params: {
          recipeProp: this.recipe,
        },
      });
    },
    handleDialog() {
      this.dialog = !this.dialog;
    },
    goBack() {
      this.$router.push({
        name: "index",
      });
    },
  },
};
</script>

<style scoped lang="scss">
.horizontal-scroll {
  overflow-x: scroll;
  white-space: nowrap;
  display: inline;
  width: 100%;
  @media (min-width: 600px) {
    overflow-x: unset;
    white-space: unset;
    display: flex;
  }
}
</style>
