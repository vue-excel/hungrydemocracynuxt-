<template>
  <v-container>
    <v-row
      v-if="recipeProp"
      :style="{
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
      }"
    >
      <v-col
        cols="12"
        md="3"
        v-for="(image, index) in this.getRecipeImageUrls"
        :key="index"
      >
        {{ index.image }}
        <v-img
          height="200"
          lazy-src="/placeholder.jpeg"
          transition="img-transition"
          alt="hungry democracy recipe images"
          :src="image ? image : '/placeholder.jpeg'"
        >
          <v-btn right tile color="error" @click="deleteImage(image)" x-small>
            <v-icon left> mdi-delete</v-icon>
            Delete
          </v-btn>
          <template v-if="image" v-slot:placeholder>
            <v-row class="fill-height ma-0" align="center" justify="center">
              <v-progress-circular
                indeterminate
                color="grey lighten-5"
              ></v-progress-circular>
            </v-row>
          </template>
        </v-img>
      </v-col>
      <v-img
        v-if="uploading"
        :style="{ margin: '10px' }"
        max-height="150"
        max-width="250"
        alt="hungry democracy image uploading"
        lazy-src="/placeholder.jpeg"
        transition="img-transition"
        src="nodata"
      >
        <template v-if="uploading" v-slot:placeholder>
          <v-row class="fill-height ma-0" align="center" justify="center">
            <v-progress-circular
              indeterminate
              color="grey lighten-5"
            ></v-progress-circular>
          </v-row>
        </template>
      </v-img>
    </v-row>
    <br />
    <client-only>
      <UploadImages
        @changed="handleImages"
        :max="this.maxImages"
        maxError="Only 4 files can be uploaded"
        uploadMsg="  "
        fileError="Only Images are accepted"
        clearAll="Remove all images"
      />
    </client-only>
    <div class="text-h6 text-center placeholder-upload">
      Drop files/click here to upload recipe images
    </div>
    <v-overlay :value="this.getImageUploadingStatus">
      <v-progress-circular indeterminate color="primary" size="64">
      </v-progress-circular>
    </v-overlay>
  </v-container>
</template>

<script>
import { createNamespacedHelpers } from "vuex";

const { mapActions, mapGetters } = createNamespacedHelpers("addRecipeStore");

export default {
  data() {
    return {
      imageUrls: [],
      maxImages: 4,
      uploading: false,
    };
  },
  props: {
    recipeProp: {
      type: Object,
    },
  },
  components: {
    UploadImages: () => {
      if (process.client) {
        return import("vue-upload-drop-images");
      }
    },
  },
  computed: {
    ...mapGetters(["getRecipeImageUrls", "getImageUploadingStatus"]),
  },
  created() {
    this.imageUrls = this.recipeProp?.imageUrls;
    this.maxImages = 4 - this.recipeProp?.imageUrls?.length;
  },
  methods: {
    ...mapActions(["saveImageFiles", "deleteImageUrl"]),
    handleImages(files) {
      this.uploading = true;
      this.saveImageFiles(files);
      this.uploading = false;
    },
    async deleteImage(imageUrl) {
      const id = this.recipeProp.id;
      const recipeProp = this.recipeProp;
      try {
        await this.deleteImageUrl({ id, imageUrl, recipeProp });
      } catch (e) {}
      // this.imageUrls = this.imageUrls.filter(function (e) {
      //   return e !== imageUrl;
      // });
    },
  },
};
</script>

<style>
.placeholder-upload {
  opacity: 0.95;
  margin-top: -184px;
  margin-bottom: 50px;
  flex-direction: column;
}

.imgsPreview {
  visibility: hidden;
}
</style>
