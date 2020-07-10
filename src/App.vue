<template>
  <div>
    <div>
      <p>Upload an image to Firebase:</p>
      <input type="file" @change="previewImage" accept="image/*" />
    </div>
    <div>
      <p>
        Progress: {{ uploadValue.toFixed() + '%' }}
        <progress id="progress" :value="uploadValue" max="100"></progress>
      </p>
    </div>
    <div v-if="imageData != null">
      <img class="preview" :src="picture" />
      <br />
      <button @click="onUpload">Upload</button>
    </div>
  </div>
</template>
<script>
import firebase from 'firebase'

const firebaseConfig = {
  apiKey: process.env.VUE_APP_API_KEY,
  VUE_APP_AUTH_DOMAIN: process.env.VUE_APP_AUTH_DOMAIN,
  VUE_APP_DB_URL: process.env.VUE_APP_DB_URL,
  VUE_APP_PROJECT_ID: process.env.VUE_APP_PROJECT_ID,
  storageBucket: process.env.VUE_APP_STORAGE_BUCKET,
  messagingSenderId: process.env.VUE_APP_MESSAGING_SENDER_ID,
  appId: process.env.VUE_APP_ID
}
// Initialize Firebase
firebase.initializeApp(firebaseConfig)

export default {
  name: 'Upload',
  data() {
    return {
      imageData: null,
      picture: null,
      uploadValue: 0
    }
  },
  methods: {
    previewImage(event) {
      this.uploadValue = 0
      this.picture = null
      this.imageData = event.target.files[0]
    },

    onUpload() {
      this.picture = null
      const storageRef = firebase
        .storage()
        .ref(`${this.imageData.name}`)
        .put(this.imageData)

      storageRef.on(
        `state_changed`,
        snapshot => {
          console.log(snapshot)
          this.uploadValue =
            (snapshot.bytesTransferred / snapshot.totalBytes) * 100
        },
        error => {
          console.log(error.message)
        },
        () => {
          this.uploadValue = 100
          storageRef.snapshot.ref.getDownloadURL().then(url => {
            this.picture = url
            console.log(this.picture)
          })
        }
      )
    }
  }
}
</script>
