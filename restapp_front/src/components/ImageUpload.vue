<template>
    <div>
        <p class="mb-1"><strong style="font-size:20px;">{{title}}</strong></p>
        <b-img :key="imageUrl" :src="imageUrl" class="picture"></b-img>
        <input ref="imageInput" type="file" hidden @change="onChangeImages">
        <div class="btn-upload">  
          <b-button pill size="sm" @click="onClickImageUpload">Upload</b-button>
        </div>
    </div>
</template>

<script>
export default {
    name:'ImageUpload',
    props: ['title'],
    data (){
        return {
            imageUrl: null
        }
    },
    methods: {
      onClickImageUpload() {        
        this.$refs.imageInput.click();
      },
      onChangeImages(e) {
        var allowedExtensions =  
                    /(\.jpg|\.jpeg|\.png|\.gif)$/i; 

        const file = e.target.files[0]; // Get first index in files
        const reader = new FileReader();
        if(!allowedExtensions.exec(file.name)){
          alert('Invalid file type.');
          return;
        }
        
        reader.readAsDataURL(file);
        reader.onload = l => {
          this.imageUrl = URL.createObjectURL(file); // Create File URL
          let base64Image = l.target.result.split(',')[1];
          this.$emit('update-event', this.title, base64Image);
          this.$refs.imageInput.value = '';
        }
      }
    }
}
</script>

<style>
@import './../assets/app.css';
</style>