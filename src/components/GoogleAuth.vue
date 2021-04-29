<template>
  <button v-cloak @click.prevent="handleAuthClick" v-if="!modelValue">Authorize</button>
</template>

<script>
// tell eslint that gapi is defined elsewhere
/*global gapi*/
export default {
  name: 'GoogleAuth',
  props: {
    modelValue: Boolean
  },
  emits: [
    'update:modelValue'
  ],
  data () {
    return {
      apiKey: process.env.VUE_APP_DRIVE_API_KEY,
      clientId: process.env.VUE_APP_OAUTH_CLIENT_ID
    }
  },
  mounted: function() {
    // load in the JS Google API auth functions
    const self = this;
    let gapiScript = document.createElement('script');
    gapiScript.setAttribute('src', 'https://apis.google.com/js/api.js');
    gapiScript.onload = () => {
      gapi.load('client:auth2', self.initClient);
    };
    document.head.appendChild(gapiScript);
  },
  methods: {
    initClient: function() {
      // Array of API discovery doc URLs for APIs used by the quickstart
      const discovery = ['https://www.googleapis.com/discovery/v1/apis/drive/v3/rest'];

      // Authorization scopes required by the API; multiple scopes can be included, separated by spaces. 
      const scopes = [
          'https://www.googleapis.com/auth/drive.install', // allows app to appear in the open with list
          'https://www.googleapis.com/auth/drive.file',    // allows access to specific files via open with, and to files created by the app
          'https://www.googleapis.com/auth/drive.appdata'  // access to app-specific data (https://developers.google.com/drive/api/v3/appdata)
      ].join(' ');

      const self = this;
      gapi.client.init({
          apiKey: this.apiKey,
          clientId: this.clientId,
          discoveryDocs: discovery,
          scope: scopes
      }).then(function () {
          // Listen for sign-in state changes.
          gapi.auth2.getAuthInstance().isSignedIn.listen(self.emitSigninStatus);
          // Handle the initial sign-in state now
          self.emitSigninStatus();
      }, function (error) {
          console.error('Could not initialize the Google API');
          console.log(error);
      });
    },
    emitSigninStatus: function() {
      this.$emit('update:modelValue', gapi.auth2.getAuthInstance().isSignedIn.get());
    },
    handleAuthClick: function() {
      gapi.auth2.getAuthInstance().signIn();
    }
  }
}
</script>
