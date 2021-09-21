
<template>
  <div>
    <b-form-file
      :capture="true"
      v-model="imageFromFile"
      accept="image/*"
      placeholder="Drop an image here or click to choose a file"
      drop-placeholder="Let go now!"
      size="lg"
      class="image-upload"
    ></b-form-file>
    <!-- <input
      type="file"
      accept="image/png, image/jpeg"
    /> -->
    <cropper
      v-if="cropperVisible"
      :src="this.currImage"
      ref="cropper"
      class="cropper"
      backgroundClass="cropper-background"
      :stencil-props="{
        aspectRatio: 2 / 3,
      }"
    >
    </cropper>
    <div class="row">
      <img
        class="mx-auto"
        width="200"
        height="300"
        v-if="!cropperVisible"
        :src="this.currImage"
      />
    </div>
    <br />
    <b-row>
      <!-- col out of 12 total width -->
      <b-button class="col-sm-4 mx-auto" v-if="cropperVisible" @click="crop"
        >Crop</b-button
      >
    </b-row>
    <b-row>
      <!-- col out of 12 total width -->
      <b-button
        class="col-sm-4 mx-auto"
        v-if="!cropperVisible"
        @click="recrop()"
        >Re-Crop</b-button
      >
    </b-row>
  </div>
</template>

<script>
import { Cropper } from "vue-advanced-cropper";
import "vue-advanced-cropper/dist/style.css";

export default {
  props: ["img"],
  components: {
    Cropper,
  },
  watch :{
    imageFromFile : function (val) {
      this.getBase64(val);
    }
  },
  data() {
    return {
      currImage: this.img,
      initialImage: this.img,
      imageFromFile: null,
      coordinates: null,
      cropperVisible: true,
    };
  },
  methods: {
    recrop() {
      this.currImage = this.initialImage;
      this.cropperVisible = true;
      this.togglePhotoReady(false);
    },
    crop() {
      const { canvas } = this.$refs.cropper.getResult();
      this.currImage = canvas.toDataURL();
      this.cropperVisible = false;
      this.updateImageData();
      this.togglePhotoReady(true);
    },
    updateImageData() {
      this.$emit("updateImageData", this.currImage);
    },
    togglePhotoReady(isReady) {
      if(isReady !== undefined){
        this.$emit("togglePhotoReady", isReady);
      }else{
        this.$emit("togglePhotoReady");
      }
    },
    getBase64(file) {
      var reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = () => {
        this.initialImage = reader.result;
        this.currImage = reader.result;
        this.recrop();
      };
      reader.onerror = function (error) {
        console.log("Error: ", error);
      };
    },
  },
  mounted() {
    //console.log("img " + this.currImage);
  },
};
</script>


<style>
.image-upload{
  margin-bottom: 5px;
}

.img-center {
  display: block;
  margin-left: auto;
  margin-right: auto;
}
/* scoped styling messes with cropper */
.cropper-background {
  background: white;
}
</style>
