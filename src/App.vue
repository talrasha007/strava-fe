<template>
  <div id="app">
    <ul>
      <li v-for="r in activities" :key="r.id">
        <a :download="`${r.name}.gpx`" :href="`https://www.strava.com/activities/${r.id}/export_gpx`">{{r.name}}</a>
      </li>
    </ul>
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
      location.href = `http://www.strava.com/oauth/authorize?client_id=41160&response_type=code&redirect_uri=${location.href}&approval_prompt=force&scope=read,activity:read`;
    }

    const { data } = await axios.get(
      'https://www.strava.com/api/v3/athlete/activities',
      { headers: { Authorization: profile.token_type + ' ' + profile.access_token } }
    );
    this.$data.activities = data;
  },

  data() {
    return {
      activities: []
    }
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
