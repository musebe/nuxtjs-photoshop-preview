<template>
<div class="min-h-full flex flex-col justify-center py-12 sm:px-6 lg:px-8">

  <div class="mt-8 sm:mx-auto sm:w-full sm:max-w-md">
    <div class="bg-white py-8 px-4 shadow sm:rounded-lg sm:px-10">
      <form @submit.prevent="upload" class="space-y-6">
        <div>
          <label for="psd" class="block text-sm font-medium text-gray-700">
            Photoshop file
          </label>
          <div class="mt-1">
            <input @change="handleFile" id="psd" name="psd" type="file" accept=".psd" required class="appearance-none block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm">
          </div>
        </div>

        <div>
          <p class="text-muted text-sm text-center" v-if="uploading">Uploading... </p>
          <button v-else type="submit" class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
            Upload
          </button>
        </div>
      </form>

    </div>
  </div>

  <div class="w-4/5 mx-auto my-10 grid grid-cols-3 gap-4">
    <div class="h-full w-full flex m-5" v-for="image in images" :key="image" >
      <img :src="image" class="m-auto" />
    </div>
  </div>
</div>

</template>

<script>


export default {
  name: 'IndexPage',
  data(){
    return {
      file:null,
      uploading:false,
      cloudinaryFile:null,
      images:[]
    };
  },
  methods:{
    async handleFile(e) {
      this.file = e.target.files[0];
    },
    async readData(f) {
      return new Promise((resolve) => {
        const reader = new FileReader();
        reader.onloadend = () => resolve(reader.result);
        reader.readAsDataURL(f);
      });
    },
    async upload() {
      this.uploading = true;
      const fileData = await this.readData(this.file);
      this.cloudinaryFile = await this.$cloudinary.upload(fileData, {
        upload_preset: "default-preset",
        folder: "nuxtjs-photoshop-preview",
      });
      this.uploading = false;
      this.getLayers();
    },
    async getLayers(){
      this.images = [];
      let count = 0;
      let resp;
      do{
        count++;
        let url =  this.$cloudinary.image.url(
          `${this.cloudinaryFile.public_id}.jpg`,
          {page:count}
        );
        resp = await fetch(url);
        if(resp.ok){
          this.images.push(url);
        }
      }while(resp.ok);
      console.log(this.images);
    }
  }
}
</script>
