<template>
  <v-layout row justify-center>
    <v-dialog v-model="dialog" fullscreen hide-overlay transition="dialog-bottom-transition">
      <v-card dark>
        <v-toolbar color="blue">
          <v-btn icon color="green" @click.native="close">
            <v-icon>close</v-icon>
          </v-btn>
          <v-toolbar-title>Add New Meaning</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-toolbar-items>
            <v-btn color="green" @click.native="save">Save</v-btn>
          </v-toolbar-items>
        </v-toolbar>
        
        <br><br>
        
      {{word}}      
        <br><br><br>

      <v-textarea
            rows="20"
            row-height="20"
            id="addMeaningTextArea" v-model="meaning"
            outline
            label="Enter new meaning"
          ></v-textarea>


      </v-card>
    </v-dialog>
  </v-layout>
</template>

<script>
import axios from 'axios';
window.onkeypress = setFocus;
window.onkeyup = setFocus;
function setFocus() {
  if(document.getElementById("addMeaningTextArea"))
  document.getElementById("addMeaningTextArea").focus();
}

  export default {
    data () {
      return {
        dialog: true,
        notifications: false,
        sound: true,
        widgets: false,
        meaning:''
      }
    },
    methods : {
      close() {
        this.dialog = false;
        this.addNewMeaningDialogCloseFunc();
      },
      mounted() {
      },
      
      save() {
          if(this.meaning == '' || this.meaning.trim() == '') {
              alert('Enter valid values');
              return;
          }
          this.meaning = this.meaning.replace(/(\r\n|\n|\r)/gm,"<br>");

          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              }
          };

          axios.put("http://localhost:8082/meaningController/saveMeaning", {
              'word': this.wordId,
              "meaning": this.meaning.trim(),
          }, yourConfig).then(() => {
            this.close();
          }).catch(e => {
            console.log('ERROR save meaning : ', e)
          })
      }
    },
    props: [ 'word', 'wordId', 'addNewMeaningDialogCloseFunc', 'jwtToken' ]
  }
</script>