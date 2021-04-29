<template>
  <div v-show="!bookData" style="margin: 1rem">
    <GoogleAuth v-model="isSignedIn" />
    <button v-if="!isSignedIn" @click.prevent="fetchDemoBook" style="margin-left: 1rem">Use demo file</button>
  </div>
  <Reader v-if="bookData" :bookData="bookData" :appData="appData" />
</template>

<script>
import GoogleAuth from '@/components/GoogleAuth.vue'
import Reader from '@/components/Reader.vue'

export default {
  name: 'Home',
  components: {
    GoogleAuth,
    Reader
  },
  data () {
    return {
      isSignedIn: false,
      bookData: null,
      appData: null
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
