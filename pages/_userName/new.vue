<template>
  <v-container fluid :style="{ margin: 'auto', marginBottom: '300px' }">
    <div class="top-background hidden-xs-only" :style="{ height: '75px' }">
      <v-img height="75px" eager alt="hungrydemocracy top bar background image" src="/top_bar_bg.png"> </v-img>
    </div>
    <div class="d-flex align-center justify-space-between">
      <div class="d-flex align-center justify-start pt-2">
        <v-btn icon color="black" @click="$router.go(-1)">
          <v-icon large>mdi-chevron-left</v-icon>
        </v-btn>
        <div class="ml-2 text-h5 font-weight-bold">Add new recipe</div>
      </div>
      <div v-if="this.isDraft || this.draft">
        <v-btn
          color="white"
          outlined
          small
          class="error"
          @click="deleteDraftRecipe"
          >Delete Draft</v-btn
        >
      </div>
    </div>
    <div
      v-if="this.loading"
      class="align-center justify-center d-flex"
      style="height: 80vh"
    >
      <v-progress-circular
        indeterminate
        color="primary"
        :size="50"
      ></v-progress-circular>
    </div>
    <v-form v-else ref="addRecipeForm" v-model="addRecipeFormValid">
      <v-col sm="12" md="6">
        <v-text-field
          label="Recipe Title*"
          placeholder="Give your recipe a title, eg 'Pakodas'"
          outlined
          v-model="title"
          @input="titleBlur"
          :rules="titleFieldRules"
          value="title"
          class="mt-4"
        />
      </v-col>
      <v-col sm="12" md="6">
        <v-textarea
          label="Description"
          placeholder="Describe your recipe"
          outlined
          :rules="descFieldRules"
          type="text-area"
          value="desc"
          rows="4"
          no-resize
          v-model="desc"
          @input="descBlur"
          class="col"
        />
      </v-col>
      <v-row>
        <v-col cols="12">
          <v-stepper
            linear
            alt-labels
            elevation="0"
            class="pl-0 elevation-0"
            :value="stepCount"
            :v-model="stepCount"
          >
            <v-stepper-header
              class="elevation-0 pl-0 hidden-sm-and-down col col-md-6"
            >
              <v-stepper-step step="1" :complete="stepCount > 1">
                Ingredients</v-stepper-step
              >
              <v-divider></v-divider>
              <v-stepper-step step="2" :complete="stepCount > 2">
                Time</v-stepper-step
              >
              <v-divider></v-divider>
              <v-stepper-step step="3" :complete="stepCount > 3">
                Images</v-stepper-step
              >
            </v-stepper-header>
            <v-stepper-items>
              <v-stepper-content step="1" class="pa-0">
                <v-card
                  elevation="0"
                  class="mb-6 px-4 px-sm-0 px-md-0 elevation-0"
                  outlined
                  v-for="(section, index) in this.getAllIngredientSections"
                  :key="index"
                >
                  <v-row
                    align="center"
                    justify="space-between"
                    class="px-1 px-md-6"
                  >
                    <v-col cols="8" md="9">
                      <v-card-title
                        class="text-no-wrap text-overflow-ellipsis"
                        >{{ section.name }}</v-card-title
                      >
                    </v-col>
                    <v-col cols="4" md="3" class="text-right px-1 px-md-6">
                      <v-btn
                        color="red"
                        text
                        v-if="index !== 0"
                        @click="removeSectionAt(index)"
                      >
                        Remove
                      </v-btn>
                    </v-col>
                  </v-row>
                  <v-row class="px-md-12 px-sm-6 px-lg-6">
                    <IngredientsList
                      :key="getUpdates"
                      :sectionIndex="index"
                      ref="ingredientsList"
                    />
                  </v-row>
                </v-card>
                <v-dialog v-model="dialog" persistent max-width="600px">
                  <template v-slot:activator="{ on, attrs }">
                    <v-btn
                      :style="{ width: '100%' }"
                      large
                      outlined
                      color="primary"
                      dark
                      v-bind="attrs"
                      v-on="on"
                    >
                      + Add a new ingredient section
                    </v-btn>
                  </template>
                  <v-card class="pa-4">
                    <v-card-title class="font-weight-bold">
                      Add a new ingredient section</v-card-title
                    >
                    <v-card-text>
                      <v-text-field
                        label="Ingredient section name (Like spices, for marination, for tempering etc..) *"
                        required
                        v-model="currentSectionName"
                      ></v-text-field>
                    </v-card-text>
                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn color="darken-1" text @click="dialog = false">
                        Close</v-btn
                      >
                      <v-btn color="primary px-4" @click="addSection"
                        >Add section</v-btn
                      >
                    </v-card-actions>
                  </v-card>
                </v-dialog>
                <div class="mt-12">
                  <v-card class="mb-6" outlined elevation="0">
                    <v-row
                      align="center"
                      justify="space-between"
                      class="px-0 px-md-6"
                    >
                      <v-col cols="12">
                        <v-card-title>Directions</v-card-title>
                      </v-col>
                    </v-row>
                    <v-row class="px-0 px-md-12">
                      <DirectionsList :key="getUpdates" />
                    </v-row>
                  </v-card>
                </div>
                <div v-if="errorMessage.length > 2">
                  <div class="py-4 text-h6 error--text">{{ errorMessage }}</div>
                </div>
                <div class="text-center py-6 mt-6">
                  <v-btn
                    class="px-12"
                    color="primary"
                    @click="goToNextSection(1)"
                    >Next Step</v-btn
                  >
                </div>
              </v-stepper-content>
              <v-stepper-content step="2" class="pa-0">
                <v-card class="mb-6" outlined elevation="0">
                  <v-card-title>Time</v-card-title>
                  <RecipeTimes :timingErrors="timingErrors" />
                </v-card>
                <div class="text-center py-6 mt-6 d-flex justify-center">
                  <v-btn class="px-6 px-md-12 ma-2" @click="stepCount = 1"
                    >Previous Step</v-btn
                  >
                  <v-btn
                    class="px-6 px-md-12 ma-2"
                    color="primary"
                    @click="goToNextSection(2)"
                    >Next Step</v-btn
                  >
                </div>
              </v-stepper-content>
              <v-stepper-content step="3" class="pa-0">
                <v-card class="mb-6 pa-4" outlined elevation="0">
                  <v-card-title>Images</v-card-title>
                  <RecipeImages :recipeProp="this.recipe" />
                  <div v-if="imageError.length > 2" class="pl-6 mt-4">
                    <div class="py-4 text-h6 error--text">{{ imageError }}</div>
                  </div>
                </v-card>
                <div class="text-center mt-6 py-6 d-flex justify-center">
                  <v-btn class="px-6 px-md-12 ma-2" @click="stepCount = 2"
                    >Previous Step</v-btn
                  >
                  <v-btn
                    class="px-6 px-md-12 ma-2"
                    color="primary"
                    @click.stop="continueRecipe"
                    :disabled="this.getImageUploadingStatus"
                    >Next Step
                  </v-btn>
                </div>
              </v-stepper-content>
            </v-stepper-items>
          </v-stepper>
        </v-col>
      </v-row>
      <v-dialog v-model="finalDialog" persistent fullscreen>
        <v-card>
          <CategorisationStep
            @input="closingDialog"
            :recipeProp="this.recipe"
          />
        </v-card>
      </v-dialog>
    </v-form>
    <div class="footer-info">
      <v-alert
        v-if="this.getSavingStatus()"
        :type="
          this.getSavingStatus().indexOf('saved') > 0
            ? 'success'
            : this.getSavingStatus().indexOf('error') > 0
            ? 'error'
            : 'warning'
        "
        dismissible
        max-width="400px"
      >
        {{ this.getSavingStatus() }}
      </v-alert>
    </div>
  </v-container>
</template>

<script>
import IngredientsList from "@/components/addrecipe/IngredientsList";
import DirectionsList from "@/components/addrecipe/DirectionsList";
import RecipeTimes from "@/components/addrecipe/RecipeTimes";
import RecipeImages from "@/components/addrecipe/RecipeImages";
import CategorisationStep from "@/components/addrecipe/CategorisationStep";
import { getRecipeFromUrlName } from "@/service/db.service";

import { createNamespacedHelpers } from "vuex";
import {
  deletePartialRecipeFromDrafts,
  getPartialRecipeFromDrafts,
} from "../../service/db.service";

const { mapActions, mapGetters } = createNamespacedHelpers("addRecipeStore");

export default {
  data() {
    return {
      recipe: {},
      addRecipeFormValid: true,
      stepCount: 1,
      title: this.getDraftRecipeTitle(),
      loading: false,
      dialog: false,
      finalDialog: false,
      timingErrors: {
        cookingTime: null,
        prepTime: null,
        serves: null,
      },
      subTitle: this.getDraftRecipeSubTitle(),
      desc: this.getDraftRecipeDesc(),
      titleFieldRules: [
        (v) => !!v || "Recipe Title is required",
        (v) => (v && v.length >= 5) || "Title must be at least 5 characters",
      ],
      descFieldRules: [
        (v) => !!v || "Description Title is required",
        (v) =>
          (v && v.length >= 10) || "Description must be at least 10 characters",
      ],
      currentSectionName: "",
      errorMessage: "",
      imageError: "",
      draft: false,
      savingStatus: null,
    };
  },
  props: ["recipeProp", "isDraft"],
  components: {
    DirectionsList,
    IngredientsList,
    RecipeTimes,
    RecipeImages,
    CategorisationStep,
  },
  methods: {
    ...mapActions([
      "savePartialRecipeData",
      "addAndSaveNewSection",
      "removeSection",
      "convertRecipeToState",
    ]),
    ...mapGetters([
      "getDraftRecipeTitle",
      "getDraftRecipeSubTitle",
      "getDraftRecipeDesc",
      "getCookingTime",
      "getPrepTime",
      "getServes",
      "getSavingStatus",
    ]),
    closingDialog(val) {
      this.finalDialog = val;
    },
    removeSectionAt(index) {
      this.removeSection(index);
    },
    async addSection() {
      await this.addAndSaveNewSection(this.currentSectionName);
      this.dialog = false;
    },
    async goToNextSection(currentSection) {
      if (currentSection === 1) {
        if (this.$refs.addRecipeForm.validate()) {
          this.errorMessage = "";
          this.stepCount = 2;
        } else {
          this.errorMessage =
            "**Please fill all required details above to proceed to next section";
        }
      }
      if (currentSection === 2) {
        if (
          this.getCookingTime() > 0 &&
          this.getPrepTime() > 0 &&
          this.getServes() > 0
        ) {
          this.timingErrors = {
            cookingTime: null,
            prepTime: null,
            serves: null,
          };
          this.stepCount = 3;
        } else {
          if (!this.getCookingTime() > 0) {
            this.timingErrors = {
              ...this.timingErrors,
              cookingTime: "A valid number Required",
            };
          }
          if (!this.getPrepTime() > 0) {
            this.timingErrors = {
              ...this.timingErrors,
              prepTime: "A valid number Required",
            };
          }
          if (!this.getServes() > 0) {
            this.timingErrors = {
              ...this.timingErrors,
              serves: "A valid number Required",
            };
          }
        }
      }
    },
    continueRecipe() {
      if (this.$refs.addRecipeForm.validate()) {
        if (this.getRecipeImageUrls?.length < 1) {
          this.imageError = "Minimum of 1 Recipe Image is Required";
        } else {
          this.finalDialog = true;
          this.imageError = "";
        }
      }
    },
    async titleBlur() {
      await this.savePartialRecipeData({ title: this.title });
    },
    async descBlur() {
      await this.savePartialRecipeData({ desc: this.desc });
    },
    async deleteDraftRecipe() {
      const id = this.recipe.id || this.recipe.draftId;
      await deletePartialRecipeFromDrafts(id);
      this.$router.go(-1);
    },
  },
  computed: {
    ...mapGetters([
      "getAllIngredientSections",
      "getUpdates",
      "getImageUploadingStatus",
      "getRecipeImageUrls",
    ]),
    show: {
      get: function () {
        return this.finalDialog;
      },
      set: function (val) {
        this.v.$touch();
        this.finalDialog = true;
      },
    },
  },
  async mounted() {
    this.loading = true;
    try {
      if (this.$route.params.draftId) {
        const draftRecipe = await getPartialRecipeFromDrafts(
          this.$route.params.draftId
        );
        this.convertRecipeToState(draftRecipe);
        this.recipe = draftRecipe;
        this.title = draftRecipe.title;
        this.desc = draftRecipe.desc;
        this.draft = true;
      } else {
        if (this.$route.params.recipeUrlName && this.$route.params.userName) {
          const recipeFromDB = await getRecipeFromUrlName(
            this.$route.params.recipeUrlName,
            this.$route.params.userName
          );
          this.convertRecipeToState(recipeFromDB);
          this.recipe = recipeFromDB;
          this.title = recipeFromDB.title;
          this.desc = recipeFromDB.desc;
          this.draft = false;
          if (!this.recipe) {
            this.$router.push({
              name: "pageNotFound",
            });
          }
        }
        // if (this.$props.recipeProp?.title) {
        //   this.recipe = this.$props.recipeProp;
        // } else {
        //   if (this.$route.params.recipeUrlName && this.$route.params.userName) {
        //     this.recipe = await getRecipeFromUrlName(this.$route.params.recipeUrlName, this.$route.params.userName);
        //     if (!this.recipe) {
        //       this.$router.push({
        //         name: 'PageNotFound'
        //       });
        //     }
        //   }
        // }
      }
    } catch (err) {
      this.$router.push({
        name: "pageNotFound",
      });
    }
    this.loading = false;
  },
};
</script>

<style>
.footer-info {
  position: fixed;
  bottom: 15px;
  right: 35px;
  z-index: 9999;
  font-size: 14px;
  font-weight: bold;
  color: #42cc8c;
}
</style>
