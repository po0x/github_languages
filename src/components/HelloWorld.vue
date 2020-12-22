<template>
  <div>

    <v-form @submit.prevent="submit" ref="form">
      <v-text-field
        v-model="user"
        label="Type a GitHub User">
      </v-text-field>
    </v-form>

    <v-alert
      v-if="displayAlert"
      border="right"
      colored-border
      type="error"
      elevation="2"
    >
      Le nom de l'utilisateur GitHub n'a pas été trouvé. Veuillez vérifier l'ortographe.
    </v-alert>

    <div class="my-2">
      <v-btn
        v-if="userFound && repos.length>0"
        class="mr-4"
        outlined
        rounded
        text
        @click="displayChart = !displayChart">
        Toggle chart
      </v-btn>

      <v-btn
        v-if="userFound && repos.length>0"
        outlined
        rounded
        text
        @click="displayRepos = !displayRepos">
        Toggle repositories
      </v-btn>

    </div>

    <p v-if="userFound && repos.length==0">No public repo found</p>

    <ul v-if="!displayChart">
      <li v-for="(bytes, language) in mergedLanguages" :key="language">{{language}} : {{bytes}} octets</li>
    </ul>

    <pie-chart v-if="userFound && displayChart" :data="chartData"></pie-chart>

    <ul v-if="displayRepos">
      <li  v-for="(repo,index) in repos" :key="index">
        <v-card
          elevation="2"
        >
          <v-card-title>{{repo.name}}</v-card-title>
          <v-card-subtitle>{{repo.clone_url}}</v-card-subtitle>

          <v-card-text>
            <p>Main language : {{repo.language}}</p>
            <p>Owner : {{repo.owner.login}}</p>
            <p>Size : {{repo.size}} Bytes</p>
          </v-card-text>

          <v-card-actions>   
            <v-spacer></v-spacer>
            <v-btn
              outlined
              rounded
              text
              @click="checkLanguage(repo.name)">
              Check languages
            </v-btn>
          </v-card-actions>
        </v-card>
      </li>
    </ul>

  </div>
</template>

<script>
import axios from 'axios';
import PieChart from "./piechart.js";

export default {
  name: 'HelloWorld',

  props: {
    msg: String
  },

  data() {
     return {
      user : "",
      error:"",
      repos : null,
      languages: [],
      userFound: false,
      displayAlert : false,
      displayChart : false,
      displayRepos : false
     }
   },

  methods: {
    submit(){
      axios
        .get('https://api.github.com/users/'+this.user+'/repos?state=closed&access_token=605ed724b91dbe70d8bb5e594de76d401cbf535e')
        .then(response => {
          this.repos = response.data
          this.userFound=true
          this.displayAlert=false
          this.displayChart=false
          this.languages=[]
        })
        .then(this.getRepoLanguages)
        .catch((error)=>{
          this.error  = error
          this.userFound=false
          this.displayAlert=true
          this.repos=null
        })
    },

    checkLanguage(repo){
      axios
          .get('https://api.github.com/repos/'+this.user+'/'+repo+'/languages?state=closed&access_token=605ed724b91dbe70d8bb5e594de76d401cbf535e')
          .then(response => (this.languages = response.data))
    },

		getRepoLanguages() {
      for (let i = 0; i < this.repos.length; i++) {
         axios
          .get('https://api.github.com/repos/'+this.user+'/'+this.repos[i].name+'/languages?state=closed&access_token=605ed724b91dbe70d8bb5e594de76d401cbf535e')
          .then(response => (this.languages.push(response.data)))
      }
    },
  },

  computed: {
		reposName() {
			return this.repos.map(repo=>repo.name)
    },
    
    mergedLanguages(){
     const result = {};

      this.languages.forEach(language => {
        for (let [key, value] of Object.entries(language)) {
          if (result[key]) {
            result[key] += value;
          } else {
            result[key] = value
          }
        }
      });
      return result
    },

    languagesLabels(){
      let array = Object.keys(this.mergedLanguages)
      return array
    },

    languagesValues(){
      let array = Object.values(this.mergedLanguages)
      return array
    },

    chartData(){
      return {
        labels: this.languagesLabels,
        datasets: [
          {
            label: "Data One",
            backgroundColor: ['#FF6633', '#FFB399', '#FF33FF', '#FFFF99', '#00B3E6', '#E6B333', '#3366E6', '#999966', '#99FF99', '#B34D4D','#80B300', '#809900', '#E6B3B3', '#6680B3', '#66991A', '#FF99E6', '#CCFF1A', '#FF1A66', '#E6331A', '#33FFCC','#66994D', '#B366CC', '#4D8000', '#B33300', '#CC80CC', '#66664D', '#991AFF', '#E666FF', '#4DB3FF', '#1AB399','#E666B3', '#33991A', '#CC9999', '#B3B31A', '#00E680', '#4D8066', '#809980', '#E6FF80', '#1AFF33', '#999933','#FF3380', '#CCCC00', '#66E64D', '#4D80CC', '#9900B3', '#E64D66', '#4DB380', '#FF4D4D', '#99E6E6', '#6666FF'],
            data: this.languagesValues
          }
        ]
      }
    }
  },

  components: {
    'pie-chart' : PieChart,
  }
}
</script>

<style scoped>
  h3 {
    margin: 40px 0 0;
  }
  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    margin-bottom: 24px;
  }

  a {
    color: #42b983;
  }
</style>
