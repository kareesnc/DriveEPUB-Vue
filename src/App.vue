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
  <DriveFile v-if="isSignedIn && docID" v-show="!bookData" ref="driveHandle"
    :docID="docID" 
    :appData="appData" 
    @fetched="recieveDriveBook"
    @savedAppData="savedAppData"
  />
  <Reader v-if="bookData" ref="readerHandle"
    :bookData="bookData" 
    :appData="appData" 
    @updateAppDataContent="updateAppDataContent"
    @saveAppDataFile="saveAppDataFile"
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
      appData: {}
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
    // Automatically save location every minute
    this.interval = setInterval(this.saveAppDataFile, 60000);
  },
  methods: {
    fetchDemoBook: function() {
      const self = this;
      var oReq = new XMLHttpRequest();
      oReq.open('GET', import.meta.env.VITE_DEMO_FILE, true);
      oReq.responseType = 'arraybuffer';
      oReq.onload = function() {
        self.bookData = oReq.response;
      };
      oReq.send();
    },
    recieveDriveBook: function(bookData, appData) {
      if(appData) {
        this.appData = appData;
      }
      this.bookData = bookData;
    },
    updateAppDataContent: function(property,value) {
      this.appData[property] = value;
    },
    saveAppDataFile: function() {
      // TODO: There seems to be a bug where sometimes the saved location isn't exactly where you were
      // Chapter starts seem ok, but not always later pages. Could it be related to increased font size?
      if(this.$refs.driveHandle) {
        console.log('Saving app data...');
        this.$refs.driveHandle.updateAppData();
      }
      else {
        console.warn('Could not save - drive not initialized.');
      }
    },
    savedAppData: function() {
      if(this.$refs.readerHandle) {
        this.$refs.readerHandle.enableSaveButton();
      }
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
