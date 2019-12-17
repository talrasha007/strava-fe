<template>
  <div id="app">
    <div class="banner">
      <div class="brand">Activities</div>
    </div>
    <div class="content">
      <h3>{{activities.length}} Activities</h3>
      <ul>
        <li v-for="r in activities" :key="r.id">
          <a :download="`${r.name}.gpx`" :href="`https://www.strava.com/activities/${r.id}/export_gpx`">{{r.name}}</a>
        </li>
      </ul>
    </div>
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
html, body {
  margin: 0;
  padding: 0;
}

.banner {
  height: 55px;
  line-height: 55px;
}

.brand {
  margin-left: 30px;
  height: 55px;
  line-height: 55px;
  font-size: 28px;
  color: #242428;
  box-shadow: 0 1px 2px rgba(0,0,0,0.025);
  border-bottom: 1px solid rgb(240, 240, 245);
}

.content {
  padding: 30px;
}
</style>
