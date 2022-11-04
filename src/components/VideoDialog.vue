<template>

<div>

    <div v-if="confirmVideoDeletionDialog">
        <DeleteConfirmationDialog :no=dontDeleteVideo :yes=videoDeleteOk :customText=customText />
    </div>



    <v-dialog v-model="dialog" persistent max-width="800px">
          <v-card>
            <v-card-title>
              <span v-if="videoSrc != ''" class="headline">
                <v-btn color="blue darken-1" text @click="play">Play</v-btn>
              <v-btn color="blue darken-1" text @click="close">Close</v-btn>
                  <v-btn color="red" @click="confirmFileDelete(videoId, wordId)">
                      <v-icon>delete_forever</v-icon>
                  </v-btn>
                <!-- https://www.npmjs.com/package/youtube-video-js -->
                <!--youtube-video width="640" height="360" src="https://youtu.be/C_kCcQRqElw" controls/-->
             </span>
            </v-card-title>
            <v-card-text>
                    <video id="videoSrcID" width="770" height="600" controls >
                      <source type="video/webm" :src="videoSrc">
                    </video>
            </v-card-text>
            <v-card-actions>

            </v-card-actions>
          </v-card>
        </v-dialog>
    </div>
</template>

<script>
import axios from 'axios';
import DeleteConfirmationDialog from './DeleteConfirmationDialog';
//import 'youtube-video-js';

  export default {
    data: () => ({
      dialog: true,
      videoSrc: '',
        confirmVideoDeletionDialog: false,
        customText: ''
    }),
    mounted() {

      //console.log('this.wordId ', this.wordId);
      //console.log('this.videoId ', this.videoId);

        let yourConfig = {
            headers: {
                Authorization: "Bearer " + this.jwtToken
            },
            params: {
                'wordOrMeaningId' : this.wordId,
                'videoId' : this.videoId
            }
        };

        axios.get("http://localhost:8082/videoController/wordVideo", yourConfig).then((response) => {
          if(response.data == '' || response.data == null || response.data == undefined) {
            this.snackbar = true
            this.dialog = false;
          } else {
            this.videoSrc = response.data;
          }
        }).catch(e => {
          console.log('ERROR wordVideo ', e)
        })
    },
      components: {
          DeleteConfirmationDialog,
      },

    methods: {
        dontDeleteVideo() {
            this.confirmVideoDeletionDialog = false
        },
        videoDeleteOk() {
            this.confirmVideoDeletionDialog = false;

            let yourConfig = {
                headers: {
                    Authorization: "Bearer " + this.jwtToken
                },
                params: {
                    'wordOrMeaningId' : this.wordId,
                    'videoId' : this.videoId
                }
            };

            axios.delete("http://localhost:8082/videoController/deleteVideo", yourConfig).then(() => {
                this.closeVideoDialog();
                this.getAllVideosForMeaningId(this.wordId);

            }).catch(e => {
                console.log('ERROR deleteWord ', e)
            })
        },
        confirmFileDelete(id) {
            this.customText = 'Are you sure?';
            this.confirmVideoDeletionDialog = true;
        },
      play() {
          let player = document.getElementById('videoSrcID');
          player.load();
          player.play();
      },
      close() {
        this.dialog = false;
        this.closeVideoDialog();
      },

    },
    props: ['wordId', 'closeVideoDialog', 'videoId', 'jwtToken', 'getAllVideosForMeaningId']
  }
</script>
