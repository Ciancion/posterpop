<template>
<div class="capture-photo">
  <label class="capture-photo__file-btn">
            Snap your poster
      <input
            id="chooseFile"
            type="file"
            accept="image/*"
            capture="environment"
            @change="onImageCaptured($event.target.name, $event.target.files)">
    </label>
  <div v-show="imgURL">
    <b-col>
      <b-button id="sendImage" v-on:click.prevent="post">Pop it</b-button>
    </b-col><br>
    <b-button id="extractedLinkButton" v-show="urlLink">
      <a name="link-extraction" id="extractedLink" v-bind:href="urlLink">{{ urlLink }}</a>
    </b-button><br>
    <img v-bind:src="imgURL" style="width:200px" id='img-imported'>
  </div>
</div>
</template>

<script>
import firebase from 'firebase'

export default {
  name: 'CapturePhoto',
  data() {
    return {
      imgURL: null,
      strImage: null,
      urlLink: null,
      uid: '',
    };
  },
  methods: {
    getBase64Image(file) {
      return new Promise((resolve, reject) => {
        const fileReader = new FileReader();
        fileReader.addEventListener('load', () => {
          resolve(fileReader.result);
        });
        fileReader.addEventListener('error', (err) => {
          reject(err);
        });
        fileReader.readAsDataURL(file);
      });
    },
    onImageCaptured(name, files) {
      this.getBase64Image(files[0])
        .then((url) => {
          this.imgURL = url;
          this.strImage = this.imgURL.replace(/^data:image\/[a-z]+;base64,/, "");
        });
    },
    post() {
      this.$http.post(`https://vision.googleapis.com/v1/images:annotate?key=AIzaSyCpaaFUFbuWmztaoTqJ-pKJ-iZHHCyICV8`, {
        requests: [{
          image: {
            content: this.strImage,
          },
          features: [{
            type: "TEXT_DETECTION",
          }]
        }]
      }).then((dataApi) => {
        this.element = dataApi.body.responses[0].textAnnotations;
        this.element.shift();
        for (var i = 0; i < this.element.length; i++) {
          if (this.element[i].description.toLowerCase().includes(".uk") || this.element[i].description.toLowerCase().includes(".com")) {
            this.urlLink = "http://" + this.element[i].description;
          };
        };
        this.addUrl(this.urlLink);
      });
    },
    addUrl(webSite) {
      this.uid = firebase.auth().currentUser.uid;
      firebase.database().ref('users/' + this.uid).push(webSite)
    },
  },
};
</script>
