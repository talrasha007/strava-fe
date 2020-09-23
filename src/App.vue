<template>
  <div id="app">
    <div class="banner">
      <div class="brand">Activities</div>
    </div>
    <div class="content">
      <h3>{{activities.length}} Activities <span class="loading" v-if="isLoading">Loading...</span></h3>
      <table class="data">
        <thead>
          <tr>
            <th>Type</th><th>Date</th><th>Title</th><th>Time</th><th>Distance</th><th/>
          </tr>
        </thead>
        <tbody>
          <tr v-for="r in activities" :key="r.id">
            <td>{{r.type}}</td>
            <td>{{r.start_date}}</td>
            <td>{{r.name}}</td>
            <td>{{r.moving_time}}</td>
            <td>{{r.distance}}</td>
            <td><a :title="r.name" :download="`${r.name}.gpx`" :href="`https://www.strava.com/activities/${r.id}/export_gpx`">download</a></td>
          </tr>
        </tbody>
      </table>
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
    const query = qs.parse(location.search.replace(/^\?/, ''));

    if (query.code && (!profile || profile.code !== query.code)) {
      // const { data } = await axios.get('https://api.i43.io/strava/oauth/get_token', { params: query });
      const { data } = await axios.get('https://3jsdgpavmc.execute-api.us-east-1.amazonaws.com/strava/oauth/get_token', { params: query });
      profile = data;
      localStorage.setItem('strava', JSON.stringify(profile));
    }

    if (!profile || profile.expires_at < Date.now() / 1000) {
      location.href = `http://www.strava.com/oauth/authorize?client_id=41160&response_type=code&redirect_uri=${location.href}&approval_prompt=force&scope=read,activity:read`;
    }

    const per_page = 200;
    for (let page = 1; page <= 5; page++) {
      const { data } = await axios.get(
        'https://www.strava.com/api/v3/athlete/activities',
        { params: { page, per_page }, headers: { Authorization: profile.token_type + ' ' + profile.access_token } }
      );

      if (data.length > 0) {
        this.$data.activities = this.$data.activities.concat(data);
      }

      if (data.length < per_page) break;
    }

    this.isLoading = false;
  },

  data() {
    return {
      isLoading: true,
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

.loading {
  margin-left: 20px;
  font-size: 13px;
  color: #777777;
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

.data {
  width: 100%;
}

th {
  background-color: #f0f0f5;
  padding: 5px;
}

td {
  text-align: center;
}
</style>
