<template>
  <v-app id="app">
      <h1 style="text-align:center;">Face Antispoofing</h1>
        <div class="main"> 
          <div class="header">
              <b-button size="sm" v-b-modal.setting-modal class="align-middle float-right">
                <b-icon icon="gear-fill" aria-hidden="true"></b-icon>
            </b-button>
            <b-modal id="setting-modal" title="Settings" ok-only :ok-disabled="!uriState" no-close-on-backdrop no-close-on-esc hide-header-close>
              <b-form-group label="Request URI" label-for="uri-input" class="mb-0">
                <b-form-input id="uri-input" v-model="requestUri" lazy-formatter :formatter="formatter" :state="uriState" aria-describedby="input-live-help input-live-feedback">
                </b-form-input>
                <b-form-invalid-feedback id="input-live-feedback">
                  Enter the request API address.
                </b-form-invalid-feedback>
                <b-form-text id="input-live-help">URI format example : 127.0.0.1:5000/test</b-form-text>
              </b-form-group>
            </b-modal>
          </div>
          <b-form-select id="type" v-model="selected" :options="options" @change="onTypeChange" style="width:300px;"></b-form-select>
          <div class="content">
            <b-card border-variant="light" class="pictureForm">
              <b-container>
                <b-col v-for="value in options[selected].types" v-bind:key="value" class="pictureBox"> 
                  <image-upload ref="imageUpload" :title="value" @update-event="onImageUpdate"></image-upload>
                </b-col>
                <b-col class="btnRest">
                    <b-button variant="dark" block pill @click="onLivenessTest">Matching</b-button>
                </b-col>
              </b-container>
            </b-card>
            <b-card header="Matching Result" border-variant="secondary" header-border-variant="secondary">
              <b-container>
                <b-col> 
                  <p class="mb-1"><strong>Matching Request Message:</strong> {{result.result_message}}</p>
                  <p class="mb-1"><strong>Fake 여부:</strong> {{result.results.fake}}</p>
                  <p class="mb-0"><strong>{{selected !== null ? options[selected].text : ''}} Score:</strong> {{result.results.score}}</p>
                </b-col>
              </b-container>
            </b-card>
          </div>
        </div>
  </v-app>
</template>

<script>
import ImageUpload from './components/ImageUpload.vue'
import ApplicationProperties from './assets/properties.json'
import "@/assets/app.css";

export default {
  components: { ImageUpload },
  name: 'App',
  data () {
    return {
        selected:0,
        options: [
          { value: 0, text: 'Color-Depth-IR', types: ['Color', 'Depth', 'IR'] },
          { value: 1, text: 'Color-Depth', types: ['Color', 'Depth'] },
          { value: 2, text: 'Color-IR', types: ['Color', 'IR'] },
        ],
        images: {
          'Color':null, 
          'Depth' :null, 
          'IR': null
        },
        result : {
          result_code: 0,
          result_message: "",
          results :
          {
            mode: 0,
            fake: null,
            score: "0"
          }
        },
        requestUri: ApplicationProperties.requestUri
    }
  },
  computed: {
      uriState() {
        let expression = /(?:[0-9]{1,3}\.){3}[0-9]{1,3}\:([0-9]{1,5})(?:\/[0-9a-zA-Z]*)*$/;
        return expression.test(this.requestUri)
      }
    },
  methods: {
    onLivenessTest (){
      var imageValid = true;

      this.$refs.imageUpload.forEach(element => {
        if(element.imageUrl == null){
           imageValid = false;
           return;
        }
      });

      if(imageValid == false){
        alert('Invalid Image. Please selete an image.')
        return;
      }

      let data = {
          mode: this.selected,
          color: this.images.Color,
          depth: this.images.Depth,
          ir: this.images.IR
      }

      let headers = {
        'Access-Control-Allow-Origin': '*',
        'Content-Type': 'application/json; charset = utf-8'
      }
      this.$http.post('http://' + this.requestUri, data, headers)
      .then((response) => {
        if(response.data.result_code == 0){
          this.result.result_message = "성공";
          this.result.results.fake = response.data.results[0].fake == 1 ? true : false;
          this.result.results.score = response.data.results[0].score;
        }
        else {
          alert("Result Message : " + response.data.result_message);
          switch(response.data.result_code){
          case -1 :
            this.result.result_message = "지원하지 않는 형식";
            break;
          case -2 :
            this.result.result_message = "추후 지원 예정";
            break;
          case 99 :
            this.result.result_message = "알 수 없는 에러";
            break;
          default :
            this.result.result_message = "알 수 없는 코드";
            break;
        }  
        }
      })
      .catch(function(error){
        alert("Face anti spoofing test error : " + error.message);
      })
    },
    formatter(value) {
      return value.toLowerCase()
    },
    onTypeChange(){
      for(var type in this.images){
        this.images[type] = null;
      }

      this.$refs.imageUpload.forEach(element => {
        element.imageUrl = null;
      });

      this.result.results.fake = null;
      this.result.results.score = 0;
    },
    onImageUpdate(title, value){
      this.images[title] = value;
    }
  },
}
</script>