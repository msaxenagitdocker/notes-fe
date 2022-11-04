<template>
  <div>
    <v-btn color="blue" text>
      Priority : &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{{item.priority}}
    </v-btn>
    <v-btn @click="plus">
        <v-icon>thumb_up</v-icon>
    </v-btn>
    <v-btn @click="minus">
      <v-icon>thumb_down</v-icon>
    </v-btn>
  </div>
</template>

<script>
import axios from 'axios';

  export default {
    data: () => ({
      dialog: true
    }),
    components: {    
    },
    methods: {
      plus() {
        // set priority in DB and refresh search results

        let url; 
        if(this.recordType === "word") {
          url = "http://localhost:8082/wordController/priorityPlus";
        } else {
          url = "http://localhost:8082/meaningController/priorityPlus";
        }

          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              },
              params: {
                  'id' : this.item.id
              }
          };

        axios.get(url, yourConfig).then(() => {
          this.refreshFn('byPriority');
        }).catch(e => {
          console.log('ERROR archive ', e)
        })

      },
      minus() {
        let url; 
        if(this.recordType === "word") {
          url = "http://localhost:8082/wordController/priorityMinus";
        } else {
          url = "http://localhost:8082/meaningController/priorityMinus";
        }

          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              },
              params: {
                  'id' : this.item.id
              }
          };
        // set priority in DB and refresh search results
        axios.get(url, yourConfig).then(() => {
          this.refreshFn('byPriority');
        }).catch(e => {
          console.log('ERROR archive ', e)
        })
      }
    },
    mounted() {

    },
    props: ['item', 'recordType', 'refreshFn', 'jwtToken']
  }
</script>

<style scoped>


</style>