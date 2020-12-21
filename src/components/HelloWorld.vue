<template>
  <div class="hello">
    <h1>{{ msg }}</h1>

    <v-text-field label="Type a GitHub User"></v-text-field>

    <ul>
      <li  v-for="(repo,index) in info" :key="index">{{repo.name}}</li>
    </ul>
    
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'HelloWorld',

  props: {
    msg: String
  },

  data() {
     return {
      info : null,
      languages: []
     }
   },

  methods: {
		getRepoLanguages() {
      console.log(this.info)
      for (let i = 0; i < this.reposName.length; i++) {
        // const repoName = this.info[i].name;
        // console.log(repoName)

        // const repoName = this.reposName[i];
        // console.log(this.reposName[i])
        
         axios
          .get('https://api.github.com/repos/mefyl/'+this.reposName[i]+'/languages')
          .then(response => (this.languages.push(response.data)))
      }
    },
  },

  mounted () {
    axios
      .get('https://api.github.com/users/mefyl/repos')
      .then(response => (this.info = response.data))
      .then(this.getRepoLanguages)
  },

  computed: {
		reposName() {
			let reposName = this.info.map(repo=>repo.name)
			return reposName
    }
	}
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
a {
  color: #42b983;
}
</style>
