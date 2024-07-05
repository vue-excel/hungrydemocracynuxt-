<template>
  <v-container>
    <v-row>
      <v-col cols="12" md="3" sm="12">
        <div style="width: 100%" class="d-flex justify-start align-center">
          <div class="pa-4">
            <v-avatar size="36px">
              <img src="/cooking_time.png" alt="hungry democracy cooking time icon" />
            </v-avatar>
          </div>
          <div class="subtitle-1 font-weight-bold text-capitalize">
            Cooking Time
          </div>
        </div>
        <div>
          <v-text-field
            v-model="cookingTime"
            type="number"
            min="0"
            step="1"
            label="Cooking time (minutes)"
            outlined
            :error-messages="timingErrors.cookingTime"
            @input="updateCookingTime"
          />
        </div>
      </v-col>
      <v-col cols="12" md="3" sm="12">
        <div style="width: 100%" class="d-flex justify-start align-center">
          <div class="pa-4">
            <v-avatar size="36px">
              <img src="/preparing_time.png" alt="hungry democracy Prep time icon" />
            </v-avatar>
          </div>
          <div class="subtitle-1 font-weight-bold text-capitalize">
            Prep Time
          </div>
        </div>
        <v-text-field
          v-model="prepTime"
          type="number"
          min="0"
          step="1"
          label="Prep time (minutes)"
          outlined
          :error-messages="timingErrors.prepTime"
          @input="updatePrepTime"
        />
      </v-col>
      <v-col cols="12" md="3" sm="12">
        <div style="width: 100%" class="d-flex justify-start align-center">
          <div class="pa-4">
            <v-avatar size="36px">
              <img src="/servings.png" alt="hungrydemocracy Serving icon" />
            </v-avatar>
          </div>
          <div class="subtitle-1 font-weight-bold text-capitalize">Serves</div>
        </div>
        <v-text-field
          v-model="serves"
          type="number"
          min="0"
          max="100"
          label="No. of Serves"
          outlined
          :error-messages="timingErrors.serves"
          @input="updateServes"
        />
      </v-col>
      <v-col md="3" />
    </v-row>
  </v-container>
</template>

<script>
import { createNamespacedHelpers } from "vuex";

const { mapActions, mapGetters } = createNamespacedHelpers("addRecipeStore");
export default {
  data() {
    return {
      cookingTime: null,
      prepTime: null,
      serves: null,
    };
  },
  props: {
    timingErrors: {
      type: Object,
    },
  },
  methods: {
    ...mapActions(["saveCookingTime", "savePrepTime", "saveServes"]),
    ...mapGetters(["getCookingTime", "getPrepTime", "getServes"]),
    updateCookingTime() {
      this.saveCookingTime(this.cookingTime);
    },
    updatePrepTime() {
      this.savePrepTime(this.prepTime);
    },
    updateServes() {
      this.saveServes(this.serves);
    },
  },
  mounted() {
    this.cookingTime = this.getCookingTime();
    this.prepTime = this.getPrepTime();
    this.serves = this.getServes();
  },
};
</script>

<style></style>
