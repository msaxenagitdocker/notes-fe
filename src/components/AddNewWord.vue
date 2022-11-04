<template>
  <v-layout row justify-center>
    <v-dialog v-model="dialog" fullscreen hide-overlay transition="dialog-bottom-transition">
      <v-card dark>
        <v-toolbar dark color="blue">
          <v-btn icon dark @click.native="close">
            <v-icon>close</v-icon>
          </v-btn>
          <v-toolbar-title>Add New Word</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-toolbar-items>
            <v-btn dark flat @click.native="save">Save</v-btn>
          </v-toolbar-items>
        </v-toolbar>
        
        <br>
        
        <v-text-field v-model="word" @keyup.enter="save" solo-inverted flat hide-details label="Enter new word" prepend-inner-icon="add"></v-text-field>
        <br>
        <v-textarea v-model="meaning" outline label="Enter its meaning" value=""></v-textarea>


      </v-card>
    </v-dialog>
  </v-layout>
</template>

<script>
import axios from 'axios';

  export default {
    data () {
      return {
        dialog: true,
        notifications: false,
        sound: true,
        widgets: false,
        word:'',
        meaning:'',
        type: ''
      }
    },
    mounted() {
      this.word = this.wordType      
    },
    methods : {

      close() {
        this.dialog = false;
        this.parentCloseFunc();
      },
      save() {

        if(this.word == '' || this.word.trim() == '') {
          alert('Enter valid values');
          return;
        }
        
        this.type = this.wordType;

          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              }
          };

        axios.post("http://localhost:8082/wordController/saveWord", {
            "word": this.word.trim(),
            "type": this.type
        }, yourConfig).then(response => {
          
          if(response.data != undefined || response.data != '') {
              
                if(this.meaning != '' && this.meaning.trim() != '') {

                    let yourConfig1 = {
                        headers: {
                            Authorization: "Bearer " + this.jwtToken
                        }
                    };

                  axios.put("http://localhost:8082/meaningController/saveMeaning", {
                      'word': response.data,
                      "meaning": this.meaning.trim(),
                  }, yourConfig1).then(() => {
                                      
                  }).catch(e => {
                    console.log('ERROR save meaning : ', e)
                  })
                }
     
              this.dialog = false;
              this.parentCloseFunc();
          }          
        }).catch(e => {
          console.log('ERROR save word : ', e)
        })
      }
    },
    props: [ 'wordType', 'parentCloseFunc', 'jwtToken' ],
  }
</script>