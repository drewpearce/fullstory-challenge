<template>
  <b-col md="auto">
    <h3>Git Issues</h3>
    <b-card v-for="issue, index in git_data" :title="issue.title" :key="index">
      <p class="card-text">
        <b-list-group>
          <b-list-group-item>User: {{ issue.user.login }}</b-list-group-item>
          <b-list-group-item>Link: {{ issue.url }}</b-list-group-item>
        </b-list-group>
        <b-button v-on:click.prevent="getStories('drew@caffdev.com')" variant="primary">Look for Stories</b-button>
      </p>
    </b-card>
  </b-col>
</template>

<script>
export default {
  data () {
    return {
      git_data: [],
      fs_issues: [],
      git_options: {
        headers: {
          'Authorization': 'token 7457cfd99023d1f32f4b9023b1a71ea153fb3c55'
        }
      },
      fs_options: {
        headers: {
          'Authorization': 'Basic QUMxUDY6UVVNeFVEWTZia2RqZUZWWFVVWkZURXhWWmpCcVozUllNbFpRY0ZsUUwycFlXVzFsWWxwRWVYcFpZblJ3TTA5a1VUMD0='
        }
      }
    }
  },
  methods: {
    getEmail: function(data_index){
      var single_issue = this.git_data[data_index];
      this.$http.get('https://api.github.com/users/' + this.git_data[data_index].user.login, this.git_options).then(function(data){
        this.git_data[data_index].email = data.body.email;
        console.log(this.git_data);
      });
    },
    getStories: function(email){
      var options = this.fs_options;
      options.parameters = {'email': email};
      /*this.$http.get('https://www.fullstory.com/api/v1/sessions', options).then(function(data){
        this.fs_issues = data.body;
      })*/
      this.fs_issue = [
    {
        "UserId": 5722646637445120,
        "SessionId": 5629499534213120,
        "CreatedTime": 1518096268,
        "FsUrl": "https://www.fullstory.com/ui/AC1P6/session/5722646637445120:5629499534213120"
    },
    {
        "UserId": 5634472569470976,
        "SessionId": 5629499534213120,
        "CreatedTime": 1518093978,
        "FsUrl": "https://www.fullstory.com/ui/AC1P6/session/5634472569470976:5629499534213120"
    }
]
console.log(this.fs_issues);
    },
    getGitIssues: function(){
      this.$http.get('https://api.github.com/repos/drewpearce/javascript-sdk/issues', this.git_options).then(function(data){
        this.git_data = data.body;
      });
    }
  },
  created () {
    let self = this;
    self.getGitIssues();
  }
}
</script>
