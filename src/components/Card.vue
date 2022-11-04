<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <v-layout align="center">

      <v-dialog v-model="showTimePickerDialog" persistent max-width="290">
      <v-time-picker v-model="timePicker" scrollable>
        <br><br>
            <v-btn text color="red" @click="cancelSetTime">
              Cancel
            </v-btn>
            <v-spacer></v-spacer>
            <v-btn text color="green" @click="saveTime">
              OK
            </v-btn>
      </v-time-picker>
    </v-dialog>
      <v-dialog v-model="audioDialog" persistent max-width="450">
      <v-card>
        <v-card-title class="headline">{{audioTitle}}</v-card-title>
        <br>
        <v-card-actions>
          <v-btn color="orange" id="recordButton" @click="record">Start</v-btn>
          <v-btn color="orange" id="stopButton" @click="stop">Stop</v-btn>
          <v-btn color="orange" id="cancelButton" @click="audioDialog = false">Cancel</v-btn>

        </v-card-actions>
      </v-card>
    </v-dialog>
      <v-dialog v-model="editAudioDialog" persistent max-width="450">
          <v-card>
              <v-card-title class="headline">{{audioTitle}}</v-card-title>
              <br>
              <v-card-actions>
                  <v-btn color="orange" id="recordButtonEdit" @click="recordEditedVoiceMemo">Start</v-btn>
                  <v-btn color="orange" id="stopButtonEdit" @click="stopEditVoiceMemo">Stop</v-btn>
                  <v-btn color="orange" id="cancelButtonEdit" @click="editAudioDialog = false">Cancel</v-btn>

              </v-card-actions>
          </v-card>
      </v-dialog>
      <v-dialog v-model="copyImageDialog" persistent max-width="450">
      <v-card>
        <v-text-field v-model="imageIdToCopy" label="Enter ImageId to add here"></v-text-field>
        <br>
        <v-card-actions>
          <v-btn color="orange" @click="copyImageAdd">Add</v-btn>
          <v-spacer></v-spacer>
          <v-btn color="orange" @click="copyImageAddAndDeleteCopiedImage">Add and Delete Copied Image</v-btn>
          <v-spacer></v-spacer>
          <v-btn color="orange" @click="copyImageDialog = false">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
      <div v-if="confirmCatagoryChangeDialog">
          <DeleteConfirmationDialog :no=dontChangeCategory :yes=categoryChangeOk :customText=customText />
      </div>
      <div v-if="showProgress">
          <ShowProgressComponent></ShowProgressComponent>
      </div>
      <div v-if="showMeaningsDialog">
          <ShowMeanings :word=word :wordId=wordId :parentCloseFunc="close" :jwtToken=jwtToken />
      </div>
      <div v-if="confirmAudioDeleteDialog">
          <DeleteConfirmationDialog :no=dontDeleteRecording :yes=deleteRecording :customText=customText />
      </div>
      <div v-if="confirmDeletionDialog">
          <DeleteConfirmationDialog :no =dontDelete :yes=deleteOk :customText=customText />
      </div>
      <div v-if="confirmVideoDeletionDialog">
          <DeleteConfirmationDialog :no =dontDelete :yes=videoDeleteOk :customText=customText />
      </div>
      <div v-if="videoDialog">
          <VideoDialog :wordId=wordId :videoId=currentVideoId :closeVideoDialog="closeVideoDialog" :jwtToken=jwtToken />
      </div>

      <div class="ex1">
            <div v-for="item in responseList" :key="item.id">
                <v-card :id="item.id">
                   <v-container>
                      <v-layout>
                        <v-flex>
                            <div align="left" id="wordContainer" class="headline">
                              {{item.word}}
                            </div>
                        </v-flex>
                      </v-layout>
                    </v-container>
                    <hr>

                    <div v-if="selectedWordType == 'TODO'">
                        <v-btn color="red" @click="setTime(item.id)">
                          Set Time
                        </v-btn>
                        show existing time from DB, if any
                    </div>
                    <br>
                    <div>
                        <table width="100%">
                            <tr>
                                <td>
                                    <div v-if="item.meaningAvailable">
                                        <v-btn color="orange" @click="loadMeanings(item.id, item.word)"><v-icon>dehaze</v-icon></v-btn>
                                        {{item.meaningCount}}
                                    </div>
                                    <div v-else>
                                        <v-btn color="green" @click="loadMeanings(item.id, item.word)"><v-icon>add_box</v-icon></v-btn>
                                    </div>
                                </td>
                                <td align="right">
                                    <PriorityComponent :recordType="recordType" :item=item :refreshFn="refreshSearchFn" :jwtToken=jwtToken ></PriorityComponent>
                                </td>
                            </tr>
                        </table>
                    </div>

                    <br><br>
                        <table border="1" width="100%">
                            <tr>
                                <td>Videos</td>
                                <td>Audios</td>
                                <td>Attachments</td>
                            </tr>
                            <tr>
                                <td>
                                    <div v-if="item.videoAvailable">
                                        <div v-for="item1 in item.videoList" :key="item1.videoId">
                                            <v-tooltip bottom>
                                                <template v-slot:activator="{ on, attrs }">
                                                    <v-icon v-bind="attrs" v-on="on" color="red" @click="confirmVideoDelete(item.id, item1.videoId)">delete_forever</v-icon>
                                                </template>
                                                <span>Delete</span>
                                            </v-tooltip>

                                            &nbsp;
                                            <v-tooltip bottom>
                                                <template v-slot:activator="{ on, attrs }">
                                                    <v-icon v-bind="attrs" v-on="on" color="green" @click="wordVideo(item.id, item1.videoId)">play_circle_filled</v-icon>
                                                </template>
                                                <span>Play</span>
                                            </v-tooltip>

                                            &nbsp;-&nbsp; {{item1.description}}
                                        </div>
                                    </div>
                                </td>
                                <td>
                                    <div v-if="item.audioAvailable">
                                        <div v-for="item1 in item.audioList" :key="item1.audioId">
                                            <v-tooltip bottom>
                                                <template v-slot:activator="{ on, attrs }">
                                                    <v-icon v-bind="attrs" v-on="on" color="green" v-bind:id="'recordVoice-'+item1.audioId" @click="voiceMemo(item.id, item1.audioId)">play_circle_filled</v-icon>
                                                </template>
                                                <span>Play</span>
                                            </v-tooltip>
                                            &nbsp;
                                            <v-tooltip bottom>
                                                <template v-slot:activator="{ on, attrs }">
                                                    <v-icon v-bind="attrs" v-on="on" color="red" @click="deleteVoiceMemo(item.id, item1.audioId)">delete_forever</v-icon>
                                                </template>
                                                <span>Delete</span>
                                            </v-tooltip>

                                            &nbsp;
                                            <v-tooltip bottom>
                                                <template v-slot:activator="{ on, attrs }">
                                                    <v-icon v-bind="attrs" v-on="on" color="orange" v-bind:id="'editVoice-'+item1.audioId" @click="editVoice(item.id, item1.audioId)">edit</v-icon>
                                                </template>
                                                <span>Edit</span>
                                            </v-tooltip>

                                        </div>
                                    </div>
                                    <br>
                                    <v-tooltip bottom>
                                        <template v-slot:activator="{ on, attrs }">
                                            <v-icon v-bind="attrs" v-on="on" color="orange" @click="recordVoiceMemo(item.id)">record_voice_over</v-icon>
                                        </template>
                                        <span>Record voice note</span>
                                    </v-tooltip>

                                </td>
                                <td></td>
                            </tr>
                        </table>
                    <br>
                    <div align="center">
                        <v-btn color="orange" @click="copyImage(item.id)">Add Image</v-btn>
                    </div>

                    <br>

                    <WordImage :jwtToken=jwtToken :responseList=responseList :wordId=item.id :loadMeaningsFn="loadMeanings" :currentWord="item.word"/>
                
                  <br>
                    &nbsp;&nbsp;Current Category : &nbsp;&nbsp;&nbsp;&nbsp; {{item.type}}
                    <br>
                  <v-card-actions>

                    Change Category : &nbsp;&nbsp;&nbsp;&nbsp;
                    <v-select dense outlined v-model="newWordType" @change="changeCategory(item.id)"
                    :items="menuItems">
                    </v-select>

                    <v-spacer></v-spacer>

                    <v-btn color="red" id="deleteButton" @click="confirmDeletion(item.id)">Delete</v-btn>


                  </v-card-actions>
                  
                </v-card>
                <br>

              </div>

    </div>
  </v-layout>
</template>

<script>
import axios from 'axios';
import ShowMeanings from './ShowMeanings';
import WordImage from './WordImage';
import DeleteConfirmationDialog from './DeleteConfirmationDialog';
import VideoDialog from './VideoDialog';
import PriorityComponent from './PriorityComponent';
import ShowProgressComponent from './ShowProgressComponent';

let recorder;
let audioBlob;

const recordAudio = () =>
  new Promise(async resolve => {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    const mediaRecorder = new MediaRecorder(stream);
    const audioChunks = [];

    mediaRecorder.addEventListener("dataavailable", event => {
      audioChunks.push(event.data);
    });

    const start = () => mediaRecorder.start();

    const stop = () =>
      new Promise(resolve => {
        mediaRecorder.addEventListener("stop", () => {
          audioBlob = new Blob(audioChunks);
          const audioUrl = URL.createObjectURL(audioBlob);
          const audio = new Audio(audioUrl);
          const play = () => audio.play();
          resolve({ audioBlob, audioUrl, play });
        });

        mediaRecorder.stop();
      });

    resolve({ start, stop });
  });

const startRecording = async () => {
  recorder = await recordAudio();
  recorder.start();
};

const stopEditVoiceRecording = async (wordId, responseList, audioId, jwtToken) => {
    //myCurrentAudio = await recorder.stop();
    await recorder.stop();
    let reader = new FileReader();
    let base64data;
    reader.readAsDataURL(audioBlob);
    reader.onloadend = async function () {
        base64data = reader.result;

        let yourConfig = {
            headers: {
                Authorization: "Bearer " + jwtToken
            }
        };

        await axios.post("http://localhost:8082/audioController/saveOrUpdateAudio", {
            'refId': wordId,
            'audio': base64data,
            'audioId': audioId
        }, yourConfig)
            .then(response => {
                // JSON responses are automatically parsed.
                let base64_decode = new Buffer(response.data, 'base64');

                const audioChunks = [];
                audioChunks.push(base64_decode);

                let audioBlob = new Blob(audioChunks);
                let audioUrl = URL.createObjectURL(audioBlob);
                let audio = new Audio(audioUrl);
                audio.play();


                for (let i = 0; i < responseList.length; i++) {
                    if (responseList[i].id === wordId) {

                        let yourConfig1 = {
                            headers: {
                                Authorization: "Bearer " + jwtToken
                            },
                            params: {
                                'refId': wordId
                            }
                        };

                        axios.get("http://localhost:8082/audioController/audioByRefIdOrderByDateTime", yourConfig1).then(response => {
                            //console.log('this.responseList ', responseList);
                            //console.log('response.data ', response.data);

                            if (response.data.length != 0) {
                                responseList[i].audioAvailable = true;
                                responseList[i].audioList = response.data;
                            } else {
                                responseList[i].audioAvailable = false;
                            }
                            //this.showProgress = false;
                        }).catch(e => {
                            console.log('ERROR 333333333 : ', e)
                        });
                        break;
                    }
                }
            })
            .catch(e => {
                alert('ERRORRORORROOR 133 ', e)
            })

    }
};

const stopRecording = async (wordId, responseList, jwtToken) => {
  //myCurrentAudio = await recorder.stop();
  await recorder.stop();
  let reader = new FileReader();
    let base64data;
 reader.readAsDataURL(audioBlob); 
 reader.onloadend = async function () {
     base64data = reader.result;

     let yourConfig = {
         headers: {
             Authorization: "Bearer " + jwtToken
         }
     };

     await axios.post("http://localhost:8082/audioController/saveOrUpdateAudio", {
         'refId': wordId,
         'audio': base64data
     }, yourConfig)
         .then(response => {
             // JSON responses are automatically parsed.
             let base64_decode = new Buffer(response.data, 'base64');

             const audioChunks = [];
             audioChunks.push(base64_decode);

             let audioBlob = new Blob(audioChunks);
             let audioUrl = URL.createObjectURL(audioBlob);
             let audio = new Audio(audioUrl);
             audio.play();


             for (let i = 0; i < responseList.length; i++) {
                 if (responseList[i].id === wordId) {
                     let yourConfig1 = {
                         headers: {
                             Authorization: "Bearer " + jwtToken
                         },
                         params: {
                             'refId': wordId
                         }
                     };
                     axios.get("http://localhost:8082/audioController/audioByRefIdOrderByDateTime", yourConfig1).then(response => {
                         //console.log('this.responseList ', responseList);
                         //console.log('response.data ', response.data);

                         if (response.data.length != 0) {
                             responseList[i].audioAvailable = true;
                             responseList[i].audioList = response.data;
                         } else {
                             responseList[i].audioAvailable = false;
                         }
                         //this.showProgress = false;
                     }).catch(e => {
                         console.log('ERROR 333333333 : ', e)
                     });
                     break;
                 }
             }
         })
         .catch(e => {
             alert('ERRORRORORROOR 1 ', e)
         })

 }
};

function playRecording(audioData) {
  let base64_decode = new Buffer(audioData, 'base64');
  const audioChunks = [];
  audioChunks.push(base64_decode);
    let audioBlob = new Blob(audioChunks);
    let audioUrl = URL.createObjectURL(audioBlob);
    let audio = new Audio(audioUrl);
    audio.play();
    console.log('done');
}



  export default {
    data: () => ({
      videoDialog: false,
      videoSrc: '',
      audioTitle: '',
      wordId: '',
      curentAudioId: '',
      word: '',
      showMeaningsDialog: false,
      meaningList:[],
      confirmDeletionDialog: false,
        customText: '',
      confirmVideoDeletionDialog: false,
      confirmAudioDeleteDialog: false,
      audioDialog: false,
        editAudioDialog: false,
      newWordType: '',
      confirmCatagoryChangeDialog: false,
      catagoryChangeWordId: '',

      recordType: 'word',
      copyImageDialog: false,
      imageIdToCopy: '',
      currentIdToAddImage: '',
      showProgress: false,

      showTimePickerDialog: false,
      timePicker: null,
      settingTimeForId: '',
      currentVideoId: '',
    }),
    components: {    
      ShowProgressComponent,
      ShowMeanings,
      WordImage,
      DeleteConfirmationDialog,
      VideoDialog,
      PriorityComponent,
    },
    beforeUpdate() {
    },
    beforeMount() {

    },
    mounted() {
        console.log('current page ::: Card.vue');
      this.newWordType = this.selectedWordType;
    },
    created() {
      //console.log('responseList ', this.responseList);
    },
    methods: {
      setTime(id) {
        this.settingTimeForId = id;
        console.log('setting time for id ', this.settingTimeForId);
        this.showTimePickerDialog = true;
      },
      cancelSetTime() {
        console.log('dont set time for ', this.settingTimeForId);
        this.showTimePickerDialog = false;
      },
      saveTime() {
        console.log('saving time for id ', this.settingTimeForId);
        console.log('timePicker ', this.timePicker);
        this.showTimePickerDialog = false;
      },
      copyImage(id) {
        this.currentIdToAddImage = id;
        this.copyImageDialog = true;
      },
      copyImageAdd() {
        this.showProgress = true;
          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              },
              params: {
                  'imageId' : this.imageIdToCopy,
                  'id': this.currentIdToAddImage
              }
          };

        axios.get("http://localhost:8082/imageController/copyImageTo", yourConfig).then(() => {
          this.imageIdToCopy = '';
          this.copyImageDialog = false;
          this.showProgress = false;
          this.refreshSearchFn();
        }).catch(e => {
          console.log('ERROR archive ', e)
        })        
      },
      copyImageAddAndDeleteCopiedImage() {
        this.showProgress = true;

          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              },
              params: {
                  'imageId' : this.imageIdToCopy,
                  'id': this.currentIdToAddImage
              }
          };

        axios.get("http://localhost:8082/imageController/copyImageToAndDelete", yourConfig).then(() => {
          this.imageIdToCopy = '';
          this.copyImageDialog = false;
          this.showProgress = false;
          this.refreshSearchFn();
        }).catch(e => {
          console.log('ERROR archive ', e)
        })        
      },
      categoryChangeOk() {
          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              }
          };

        axios.post("http://localhost:8082/wordController/changeWordCategory", {
            'id' : this.catagoryChangeWordId,
            'type' : this.newWordType
        }, yourConfig).then(response => {
          this.confirmCatagoryChangeDialog = false;
          this.newWordType = '';
          this.refreshSearchFn();
        }) 
        .catch(e => {
          this.confirmCatagoryChangeDialog = false;
          console.log('ERROR 2  ;;;;;;;;;;;  ', e);
        })
      },
      dontChangeCategory() {
        this.confirmCatagoryChangeDialog = false;
      },
      changeCategory(wordId) {
        this.catagoryChangeWordId = wordId;
        this.customText = 'Are you sure?';
        this.confirmCatagoryChangeDialog = true;
      },
      wordVideo(wordMeaningId, videoId) {
        this.wordId = wordMeaningId;
        this.currentVideoId = videoId;
        this.videoDialog = true;
      },
      closeVideoDialog() {
        this.videoDialog = false;
      },
    loadMeanings(wordId, word) {
      this.wordId = wordId;
      this.word = word;
      this.showMeaningsDialog=true
    },
    confirmVideoDelete(wordMeaningId, videoId) {
      this.wordId = wordMeaningId;
      this.currentVideoId = videoId;
        this.customText = 'Are you sure?';
      this.confirmVideoDeletionDialog = true
    },
    confirmDeletion(wordId) {
      this.wordId = wordId;
      this.customText = 'Are you sure? It will remove all data related to this item';
      this.confirmDeletionDialog = true
    },
    dontDelete() {
      this.confirmDeletionDialog = false;
      this.confirmVideoDeletionDialog = false;
      document.getElementById("wordContainer").contentEditable = false;
      this.wordId = ''
    },
    videoDeleteOk() {
      this.confirmVideoDeletionDialog = false;
        let yourConfig = {
            headers: {
                Authorization: "Bearer " + this.jwtToken
            },
            params: {
                'wordOrMeaningId' : this.wordId,
                'videoId' : this.currentVideoId
            }
        };

      axios.delete("http://localhost:8082/videoController/deleteVideo", yourConfig).then(() => {

        for(let i=0; i<this.responseList.length;i++) {
                      if(this.responseList[i].id == this.wordId) {
                          let yourConfig = {
                              headers: {
                                  Authorization: "Bearer " + this.jwtToken
                              },
                              params: {
                                  'wordOrMeaningId' : this.wordId
                              }
                          };
                          axios.get("http://localhost:8082/videoController/videosBywordOrMeaningId", yourConfig).then(response => {
                            if(response.data.length != 0) {
                              this.responseList[i].videoList = response.data;
                                this.responseList[i].videoAvailable = true;
                            } else {
                              this.responseList[i].videoAvailable = false
                            }
                          }).catch(e => {
                            console.log('ERROR mounted : ', e)
                          });
                        break;
                      }
                    }
      }).catch(e => {
        console.log('ERROR deleteWord ', e)
      })      
    },
    deleteOk() {
      this.confirmDeletionDialog = false;
        let yourConfig = {
            headers: {
                Authorization: "Bearer " + this.jwtToken
            },
            params: {
                'wordId' : this.wordId
            }
        };
      axios.delete("http://localhost:8082/wordController/deleteWord", yourConfig).then((response) => {
        if(response.status == 200 && this.responseList != undefined && this.responseList != '') {
          for(var i=0; i<this.responseList.length;i++) {
            if(this.responseList[i].id == this.wordId) {
              this.responseList.splice(i, 1);
              break;
            }
          }
        }
      }).catch(e => {
        console.log('ERROR deleteWord ', e)
      });
      this.parentCloseFunc();
    },
    dontDeleteRecording() {
      this.confirmAudioDeleteDialog = false;
    },
    deleteRecording() {
      this.confirmAudioDeleteDialog = false;
        let yourConfig = {
            headers: {
                Authorization: "Bearer " + this.jwtToken
            },
            params: {
                'refId' : this.wordId,
                'audioId' : this.curentAudioId
            }
        };

      axios.get("http://localhost:8082/audioController/deleteAudioById", yourConfig).then(response => {
          for(let i=0; i<this.responseList.length;i++) {
              if(this.responseList[i].id === this.wordId) {
                  let yourConfig1 = {
                      headers: {
                          Authorization: "Bearer " + this.jwtToken
                      },
                      params: {
                          'refId' : this.wordId
                      }
                  };
                  axios.get("http://localhost:8082/audioController/audioByRefIdOrderByDateTime", yourConfig1).then(response => {
                      if(response.data.length != 0) {
                          this.responseList[i].audioAvailable = true;
                          this.responseList[i].audioList = response.data;
                      } else {
                          this.responseList[i].audioAvailable = false;
                      }
                      //this.showProgress = false;
                  }).catch(e => {
                      console.log('ERROR 111 : ', e)
                  });
                  break;
              }
          }
        }).catch(e => {
          console.log('ERRORRORORROOR 3 ', e)
        })
    },
    deleteVoiceMemo(wordId, audioId) {
      this.wordId = wordId;
      this.curentAudioId = audioId;
      this.customText = 'Are you sure?';
      this.confirmAudioDeleteDialog = true;
      //document.getElementById('recordVoice-'+audioId).style.display = 'block';
    },
        recordVoiceMemo(wordId) {
            this.wordId = wordId;
            this.audioTitle = 'Click Start to record your voice note';
            this.audioDialog = true;
            document.getElementById('stopButton').style.display = 'none';
            document.getElementById('recordButton').style.display = 'block';
            document.getElementById('cancelButton').style.display = 'block';
        },
      voiceMemo(wordId, audioId) {
        this.wordId = wordId;
        this.audioTitle = 'Click Start to record your voice note';

          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              },
              params: {
                  'refId' : wordId,
                  'audioId' : audioId
              }
          };

        axios.get("http://localhost:8082/audioController/audioById", yourConfig).then(response => {
              //console.log('this current response ', response.data);
            document.getElementById('editVoice-'+audioId).style.display = 'block';
            playRecording(response.data.audio);
        }).catch(e => {
          console.log('ERRORRORORROOR 4  ', e)
        });
      },
      editVoice(wordId, audioId) {
        this.wordId = wordId;
          this.curentAudioId = audioId;
            this.editAudioDialog = true;
        this.audioTitle = 'Editing voice memo';

        document.getElementById('recordButtonEdit').style.display = 'block';
        document.getElementById('stopButtonEdit').style.display = 'none';
        document.getElementById('cancelButtonEdit').style.display = 'block';
      },
        recordEditedVoiceMemo() {
            document.getElementById('stopButtonEdit').style.display = 'block';
            document.getElementById('recordButtonEdit').style.display = 'none';
            document.getElementById('cancelButtonEdit').style.display = 'none';
            this.audioTitle = 'Recording voice memo, Click Stop once done';
            startRecording(this.wordId);
        },
      record() {
          document.getElementById('stopButton').style.display = 'block';

        document.getElementById('recordButton').style.display = 'none';
        document.getElementById('cancelButton').style.display = 'none';

        this.audioTitle = 'Recording voice memo, Click Stop once done';
        startRecording(this.wordId);
      },
      async stop() {
          this.audioDialog = false;
          this.editAudioDialog = false;
          await stopRecording(this.wordId, this.responseList, this.jwtToken);
      },
        async stopEditVoiceMemo() {
            this.audioDialog = false;
            this.editAudioDialog = false;
            await stopEditVoiceRecording(this.wordId, this.responseList, this.curentAudioId, this.jwtToken);
        },
      close() {
        this.showMeaningsDialog = false;     
        document.getElementById("wordContainer").contentEditable = false;
        this.refreshSearchFn();
      }
    },
    props: ['responseList', 'parentCloseFunc', 'selectedWordType', 'refreshSearchFn', 'menuItems', 'jwtToken']
  }
</script>

<style scoped>
  
.responsive {
    width: 100%;
    height: auto;
}

div.ex1 {
  width:50%;
  margin: auto;

}

</style>