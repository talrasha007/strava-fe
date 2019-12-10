<template>
  <div id="app">
  </div>
</template>

<script>
const qs = require('querystring');
const axios = require('axios');

export default {
  name: 'app',
  async beforeMount() {
    let profile = JSON.parse(localStorage.getItem('strava') || 'null');
    const query = qs.parse(location.search);

    if (query.code && (!profile || profile.code !== query.code)) {
      const { data } = await axios.get('/strava/oauth/get_token', { params: query });
      profile = data;
      localStorage.setItem('strava', JSON.stringify(profile));
    }

    if (!profile || profile.expires_at < Date.now() / 1000) {
      location.href = `http://www.strava.com/oauth/authorize?client_id=41160&response_type=code&redirect_uri=${location.href}&approval_prompt=force&scope=read`;
    }
  },
  components: {
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
