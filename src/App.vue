<template>
  <GoogleAuth v-model="isSignedIn" />
  <p v-if="isSignedIn">Currently signed in</p>
  <p v-if="!isSignedIn">Currently signed out</p>
</template>

<script>
import GoogleAuth from '@/components/GoogleAuth.vue'

export default {
  name: 'Home',
  components: {
    GoogleAuth
  },
  data () {
    return {
      isSignedIn: false
    }
  }
}
</script>
