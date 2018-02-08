<template>
  <div id="app">
    <header>
      <h1>{{ title }}</h1>
    </header>
    <b-container fluid>
      <b-row>
        <b-alert variant="danger" dismissible :show="show_alert" @dismissed="show_alert=false">{{ alert_text }}</b-alert>
      </b-row>
      <b-row>
        <b-alert variant="warning" dismissible :show="show_welcome_alert" @dismissed="show_welcome_alert=false">{{ welcome_alert_text }}</b-alert>
      </b-row>
      <b-row>
        <b-alert variant="success" dismissible :show="show_add_comment_alert" @dismissed="show_add_comment_alert=false"><span v-html="add_comment_alert_text">.</span></b-alert>
      </b-row>
      <b-row>
        <b-col v-if="git_data.length > 0">
          <h3>Git Issues</h3>
          <b-card v-for="issue, index in git_data" :title="issue.title" :key="index">
            <p class="card-text">
              <b-list-group>
                <b-list-group-item>User: {{ issue.user.login }}</b-list-group-item>
                <b-list-group-item><a :href="issue.url" target="_blank">Link to Issue</a></b-list-group-item>
              </b-list-group>
              <b-button v-on:click.prevent="findStories(index)" variant="primary">Look for Stories</b-button>
            </p>
          </b-card>
        </b-col>
        <b-col v-if="fs_stories.length > 0">
          <h3>Related Stories</h3>
          <b-card v-for="story, index in fs_stories" :title="String(story.SessionId)" :key="index">
            <p class="card-text">
              <b-list-group>
                <b-list-group-item>Timestamp: {{ story.CreatedTime }}</b-list-group-item>
                <b-list-group-item><a :href="story.FsUrl" target="_blank">Link to Story</a> (must be logged in to view)</b-list-group-item>
                <b-button v-on:click.prevent="addToIssue(story.FsUrl)">Add to GitHub Issue?</b-button>
              </b-list-group>
            </p>
          </b-card>
        </b-col>
      </b-row>
      <b-row>
        <b-form id="token_form">
          <h4>Enter your credentials</h4>
          <b-form-row>
            <b-col sm="3">
              <b-form-group id="fs_token_label" label="FullStory Token" label-for="fs_token"></b-form-group>
            </b-col>
            <b-col sm="9">
              <b-form-input id="fs_token" typ="password" v-model="fs_token"></b-form-input>
            </b-col>
          </b-form-row>
          <b-form-row>
            <b-col sm="3">
              <b-form-group id="gh_token_label" label="GitHub Token" label-for="gh_token"></b-form-group>
            </b-col>
            <b-col sm="9">
            <b-form-input id="gh_token" typ="password" v-model="gh_token"></b-form-input>
            </b-col>
          </b-form-row>
          <b-form-row>
            <b-button v-on:click.prevent="getGitIssues" variant="primary">Save Credentials and Get Issues</b-button>
          </b-form-row>
        </b-form>
        <b-card class="explainer">
          <p class="card-text">
            This app will take pull in data from the issues on <a href="https://github.com/drewpearce/javascript-sdk/issues" target="_blank">Javscript SDK</a> and try to correlate them with FullStory data, based upon the user's registered email address. You will need to supply a GitHub token to retrieve user email address and make comments, and a FullStory token to be able to retrieve story data.
          </p>
        </b-card>
      </b-row>
    </b-container>
  </div>
</template>

<script>

export default {
  name: 'app',
  data () {
    return {
      title: 'FullStory Coding Challenge',
      show_alert: false,
      welcome_alert_text: 'This app needs your Full Story API token and a GitHub API token to correllate data.',
      show_welcome_alert: true,
      show_add_comment_alert: false,
      add_comment_alert_text: '',
      alert_text: '',
      fs_token: '',
      gh_token: '',
      git_data: [],
      fs_stories: [],
      current_issue: 0
    }
  },
  computed: {
    git_options: function() {
      if (this.gh_token == '') {
        return {};
      } else {
        return {
          headers: {
            'Authorization': 'token ' + this.gh_token
          }
        };
      }
    },
    fs_options: function() {
      return {
        headers: {
          'Authorization': 'Basic ' + this.fs_token
        }
      };
    }
  },
  created() {

  },
  methods: {
    getStories: function(email){
      var options = this.fs_options;
      options.params = {'email': email};
      this.$http.get('https://www.fullstory.com/api/v1/sessions', options).then(function(data){
        this.fs_stories = data.body;
      }).catch(function(data){
        this.sendAlert('There was an issue retrieving the stories, or none matched your search. Please check the console for details.');
      });
    },
    findStories: function(data_index){
      this.$http.get('https://api.github.com/users/' + this.git_data[data_index].user.login, this.git_options).then(function(data){
        this.git_data[data_index].email = data.body.email;
        return data.body.email;
      }).then(function(data){
        if (data === null) {
          this.current_issue = 0;
          this.sendAlert('There was an issue retrieving the associated email, or the user\'s email is private.');
        } else {
          this.current_issue = this.git_data[data_index].number;
          this.getStories(data);
        }
      });
    },
    addToIssue: function(FsUrl){
      var payload = {
        'body': 'This issue has a FullStory: ' + FsUrl
      };
      var url = 'https://api.github.com/repos/drewpearce/javascript-sdk/issues/' + this.current_issue + 'comments'
      this.$http.post(url, payload, this.git_options).then(function(data){
        console.log(data.body);
        this.add_comment_alert_text = 'Comment successfully added: <a href="' + data.body.html_url + '" target="_blank">New Comment</a>';
        this.show_add_comment_alert = true;
      });
    },
    getGitIssues: function(){
      this.$http.get('https://api.github.com/repos/drewpearce/javascript-sdk/issues', this.git_options).then(function(data){
        this.git_data = data.body;
      });
    },
    sendAlert: function(text){
      this.alert_text = text;
      this.show_alert = true;
    }
  }
}
</script>

<style>
body {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}

#app {
  max-width: 85%;
  margin: 0 auto;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

#token_form {
  padding-top: 40px;
}

b-container {
  padding: 30px;
}

header {
  text-decoration: underline;
  width: 100%;
  padding: 20px;
}

.card {
  margin: 30px;
  max-width: 50%;
}
</style>
