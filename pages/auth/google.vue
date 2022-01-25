<template>
  <div>
    <h1>Contact Page</h1>
    <h1>Contact Page</h1>
  </div>
</template>
<script>

export default {
  name: 'GoogleAuthPage',

  created () {
    const token = this.$route.hash.split('access_token=')[1].split('&')[0]
    this.$cookies.set('token', token)
    this.$router.push('/')
  }
}
</script>
