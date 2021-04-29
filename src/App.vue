<template>
  <div v-show="!bookData" style="margin: 2rem; max-width: 750px;">
    <button @click.prevent="fetchDemoBook" style="margin-right: 1rem">Use demo file</button>
    <GoogleAuth v-model="isSignedIn" />
    <p v-if="!isSignedIn">
      You need to authorize this app before you can use it. 
      Click the "Authorize" button to get started.
      Or, you can click the demo button to view a demo.
    </p>
    <p v-if="isSignedIn && !docID">
      You are signed in, but no book is loaded. 
      If you have just authorized the app, you can close this window 
      and start opening EPUB books via the context menu in Google Drive.
      Or, you can click the demo button to view a demo.
    </p>
  </div>
  <DriveFile v-if="isSignedIn && docID" v-show="!bookData" 
    :docID="docID" 
    :appData="appData" 
    @fetched="recieveDriveBook" 
  />
  <Reader v-if="bookData" 
    :bookData="bookData" 
    :appData="appData" 
  />
</template>

<script>
import GoogleAuth from '@/components/GoogleAuth.vue'
import DriveFile from '@/components/DriveFile.vue'
import Reader from '@/components/Reader.vue'

export default {
  name: 'Home',
  components: {
    GoogleAuth,
    DriveFile,
    Reader
  },
  data () {
    return {
      isSignedIn: false,
      docID: null,
      bookData: null,
      appData: null
    }
  },
  mounted: function() {
    // Fetch the state, which will contain the document ID
    var params = URLSearchParams && new URLSearchParams(document.location.search.substring(1));
    var state = JSON.parse(params.get('state'));
    if (state) {
      this.docID  = state.ids[0];
    }
    else {
      console.warn('Could not get document ID from URL state');
    }
  },
  methods: {
    fetchDemoBook: function() {
      const self = this;
      var oReq = new XMLHttpRequest();
      oReq.open('GET', process.env.VUE_APP_DEMO_FILE, true);
      oReq.responseType = 'arraybuffer';
      oReq.onload = function() {
        self.bookData = oReq.response;
      };
      oReq.send();
    },
    recieveDriveBook: function(bookData, appData) {
      console.log(appData);
      this.bookData = bookData;
    }
  }
}
</script>

<style>
  html, body, #app {
    height: 100%;
    margin: 0;
    font-family: Helvetica,Arial,sans-serif;
    font-size: 1rem;
    font-weight: 400;
    line-height: 1.5;
    overflow: hidden;
  }
  button:hover {
    cursor: pointer;
  }
</style>
