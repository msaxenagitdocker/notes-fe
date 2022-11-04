<template>
  <v-layout row justify-center>
      <div v-if="confirmFileDeletionDialog">
          <DeleteConfirmationDialog :no=dontDeleteFile :yes=fileDeleteOk :customText=customText />
      </div>
      <div v-if="showProgress">
          <ShowProgressComponent></ShowProgressComponent>
      </div>
      <div v-if="confirmDeletionDialog">
      <DeleteConfirmationDialog :no=dontDelete :yes=deleteOk :customText=customText />
    </div>
      <div v-if="confirmAudioDeleteDialog">
          <DeleteConfirmationDialog :no=dontDeleteRecording :yes=deleteRecording :customText=customText />
      </div>
      <div v-if="addNewMeaningDialog">
          <AddNewMeaning :word=word :wordId=wordId :addNewMeaningDialogCloseFunc="addNewMeaningDialogClose" :jwtToken=jwtToken />
      </div>

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
      <v-dialog v-model="audioDialog" persistent max-width="450">
          <v-card>
            <v-card-title class="headline">{{audioTitle}}</v-card-title>
            <br>
            <v-card-actions>
              <v-btn color="orange" id="recordButton" @click="startRecordingMemo">Start</v-btn>
              <v-btn color="orange" id="recordingStopButton" @click="stop">Stop</v-btn>
              <v-btn color="orange" id="cancelRecordingButton" @click="audioDialog = false">Cancel</v-btn>
            </v-card-actions>
          </v-card>
      </v-dialog>
      <v-dialog v-model="editAudioDialog" persistent max-width="450">
          <v-card>
              <v-card-title class="headline">{{audioTitle}}</v-card-title>
              <br>
              <v-card-actions>
                  <v-btn color="orange" id="editRecordButton" @click="startEditingVoiceMemo">Start</v-btn>
                  <v-btn color="orange" id="editRecordingStopButton" @click="stopEditingVoiceMemo">Stop</v-btn>
                  <v-btn color="orange" id="editCancelRecordingButton" @click="editAudioDialog = false">Cancel</v-btn>
              </v-card-actions>
          </v-card>
      </v-dialog>
      <v-dialog v-model="editWordDialog" persistent max-width="600">
          <v-card>

            <br><br>
            <v-textarea
              outline
              name="input-7-4"
              label="Edit this word"
              id="editWordTextArea"
              :value=editedWord
            ></v-textarea>
            <v-card-actions>
                <v-btn color="orange" id="saveWordButton" @click.native="saveWord">Save</v-btn>
                <v-btn color="orange" @click="editWordDialog = false">Cancel</v-btn>
            </v-card-actions>
          </v-card>
      </v-dialog>
      <v-dialog v-model="editMeaningDialog" persistent max-width="900">
          <v-card>
            <br><br>
            <v-textarea rows="20" row-height="20" outline name="input-7-4" label="Edit this meaning" id="editMeaningTextArea"
                        :value=editedMeaning></v-textarea>
            <v-card-actions>
                <v-btn color="orange" id="saveMeaningButton" @click.native="saveMeaning">Save</v-btn>
                <v-btn color="orange" @click="editMeaningDialog = false">Cancel</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

      <v-dialog v-model="dialog" fullscreen hide-overlay>
        <v-card dark>
            <v-toolbar dark color="blue">
              <v-toolbar-title></v-toolbar-title>
              <v-spacer></v-spacer>
              <v-btn dark @click.native="getAllMeaningsByWordId('byDate')">
                Refresh
              </v-btn>
              <v-spacer></v-spacer>
              <v-btn dark @click.native="getAllMeaningsByWordId('byDate')">
                Sort By Date
              </v-btn>
              <v-spacer></v-spacer>
              <v-btn color="green" @click.native="addNewMeaning">
                Add new meaning
              </v-btn>
              <v-spacer></v-spacer>
              <v-btn dark @click.native="getAllMeaningsByWordId('byPriority')">
                Sort By Priority
              </v-btn>
              <v-spacer></v-spacer>
              <v-btn icon dark @click.native="close">
                <v-icon>close</v-icon>
              </v-btn>
            </v-toolbar>
            <br>
              <h1 align="center" id="showWordHere"></h1>
            <br>
            <v-btn color="orange" id="editWordButton" @click.native="editWord(wordId)">Edit Word</v-btn> &nbsp; {{meaningsList.length}}
            <br><br><br><br>
            <ul>
                  <li v-for="item in meaningsList" :key="item.id">
                      <div class="headline" v-html="intro(item.meaning)"></div>
                      <br>
                      <table width="100%">
                          <tr>
                              <td>
                                  <v-btn color="orange" @click="editMeaning(item.id, item.meaning)">
                                      <v-icon>edit</v-icon>
                                  </v-btn>
                              </td>
                              <td>
                                  <v-btn color="red" @click="deleteMeaning(item.id)">
                                      <v-icon>delete_forever</v-icon>
                                  </v-btn>
                              </td>
                              <td align="right">
                                  <PriorityComponent :recordType="recordType" :jwtToken=jwtToken :item=item :refreshFn="getAllMeaningsByWordId"></PriorityComponent>
                              </td>
                          </tr>
                      </table>
                      <br><br>
                      <table border="1" width="100%">
                          <tr>
                              <td>Videos</td>
                              <td>Audios</td>
                              <td>Attachments</td>
                          </tr>
                          <tr>
                              <td>
                                  <v-card-actions :id=item.id>
                                      <div v-if="item.videoAvailable">
                                          <ul>
                                              <li v-for="item1 in item.videoList" :key="item1.videoId">
                                                  {{item1}}
                                              </li>
                                          </ul>
                                      </div>
                                  </v-card-actions>
                              </td>
                              <td>
                                  <v-card-actions :id=item.id>
                                      <div v-if="item.audioAvailable">
                                          <ul>
                                              <li v-for="item1 in item.audioList" :key="item1.audioId">
                                                  <v-btn color="orange" v-bind:id="'recordVoice-'+item1.audioId" @click="play(item.id, item1.audioId)">Voice Memo</v-btn>
                                                  <v-btn color="red" @click="deleteRecordingButton(item.id, item1.audioId)">Delete Recording</v-btn>
                                                  <v-btn color="orange" v-bind:id="'editVoice-'+item1.audioId" @click="editRecordingButton(item.id, item1.audioId)">Edit Voice</v-btn>
                                                  - Description
                                              </li>
                                          </ul>
                                      </div>
                                  </v-card-actions>
                                  <v-btn color="orange" @click="recordVoiceMemo(item.id)">
                                      <v-icon>record_voice_over</v-icon>
                                  </v-btn>
                              </td>
                              <td>
                                  <v-card-actions :id=item.id>
                                      <div v-if="item.isAttachedFilesAvailable">
                                          <div v-for="item1 in item.attachedFileList" :key="item1.id">
                                              <v-btn color="red" @click="confirmFileDelete(item1.id, item1.linkId)">
                                                  <v-icon>delete_forever</v-icon>
                                              </v-btn>
                                              <v-btn color="green" @click="downloadItem(item1.id, item1.linkId)">
                                                  <v-icon>cloud_download</v-icon>
                                              </v-btn>
                                              &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {{item1.fileName}}
                                          </div>
                                      </div>
                                  </v-card-actions>
                              </td>
                          </tr>
                      </table>
                      <br>
                      <div align="center">
                          <v-btn color="green" @click="copyImage(item.id)">Add Image</v-btn>
                      </div>
                      <MeaningImage :meaning=item :addNewMeaningFn=addNewMeaning :closeFn=close :jwtToken=jwtToken />
                      <br>
                      <hr>
                      <br><br>
                  </li>
             </ul>
        </v-card>
      </v-dialog>

  </v-layout>
</template>

<script>

import axios from 'axios';
import AddNewMeaning from './AddNewMeaning'
import DeleteConfirmationDialog from './DeleteConfirmationDialog';
import MeaningImage from './MeaningImage';
import ShowProgressComponent from './ShowProgressComponent';
import PriorityComponent from './PriorityComponent';

let recorder;
let audioBlob;
//var myCurrentAudio;

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

const stopRecording = async (meaningId, meaningsList, jwtToken) => {
  //myCurrentAudio = await recorder.stop();
  await recorder.stop();
  let reader = new FileReader();
 let base64data;
 reader.readAsDataURL(audioBlob); 
 reader.onloadend = function() {
      base64data = reader.result;

     let yourConfig = {
         headers: {
             Authorization: "Bearer " + jwtToken
         }
     };

      axios.post("http://localhost:8082/audioController/saveOrUpdateAudio", {
          'refId' : meaningId,
          'audio' : base64data
      }, yourConfig)
        .then(response => {
          // JSON responses are automatically parsed.
          //console.log('response.data : ', response.data);
          let base64_decode = new Buffer(response.data, 'base64');

          const audioChunks = [];
          audioChunks.push(base64_decode);

            let audioBlob = new Blob(audioChunks);
            let audioUrl = URL.createObjectURL(audioBlob);
            let audio = new Audio(audioUrl);
          audio.play();


            for (let i = 0; i < meaningsList.length; i++) {
                if (meaningsList[i].id === meaningId) {

                    let yourConfig1 = {
                        headers: {
                            Authorization: "Bearer " + jwtToken
                        },
                        params: {
                            'refId': meaningId
                        }
                    };
                    axios.get("http://localhost:8082/audioController/audioByRefIdOrderByDateTime", yourConfig1).then(response => {
                        //console.log('response.data ', response.data);

                        if (response.data.length != 0) {
                            meaningsList[i].audioAvailable = true;
                            meaningsList[i].audioList = response.data;
                        } else {
                            meaningsList[i].audioAvailable = false;
                        }
                        //this.showProgress = false;
                    }).catch(e => {
                        console.log('ERROR saving audio ref : ', e)
                    });
                    break;
                }
            }
        }) 
        .catch(e => {
          console.log('ERROR saving audio ', e);
        })

  }
};

const stopEditRecording = async (meaningId, audioId, meaningsList, jwtToken) => {
    //myCurrentAudio = await recorder.stop();
    await recorder.stop();
    let reader = new FileReader();
    let base64data;
    reader.readAsDataURL(audioBlob);
    reader.onloadend = function() {
        base64data = reader.result;

        let yourConfig = {
            headers: {
                Authorization: "Bearer " + jwtToken
            }
        };
        axios.post("http://localhost:8082/audioController/saveOrUpdateAudio", {
            'refId' : meaningId,
            'audioId': audioId,
            'audio' : base64data
        }, yourConfig)
            .then(response => {
                // JSON responses are automatically parsed.
                //console.log('response.data : ', response.data);
                let base64_decode = new Buffer(response.data, 'base64');

                const audioChunks = [];
                audioChunks.push(base64_decode);

                let audioBlob = new Blob(audioChunks);
                let audioUrl = URL.createObjectURL(audioBlob);
                let audio = new Audio(audioUrl);
                audio.play();


                for (let i = 0; i < meaningsList.length; i++) {
                    if (meaningsList[i].id === meaningId) {
                        let yourConfig2 = {
                            headers: {
                                Authorization: "Bearer " + jwtToken
                            },
                            params: {
                                'refId': meaningId
                            }
                        };
                        axios.get("http://localhost:8082/audioController/audioByRefIdOrderByDateTime", yourConfig2).then(response => {
                            //console.log('response.data ', response.data);

                            if (response.data.length != 0) {
                                meaningsList[i].audioAvailable = true;
                                meaningsList[i].audioList = response.data;
                            } else {
                                meaningsList[i].audioAvailable = false;
                            }
                            //this.showProgress = false;
                        }).catch(e => {
                            console.log('ERROR EDIT AUDIO GETTING DATA BACK: ', e)
                        });
                        break;
                    }
                }
            })
            .catch(e => {
                alert('ERROR WHILE EDITING AUDIO', e)
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

}

  export default {
    data () {
      return {
        dialog: true,
        addNewMeaningDialog: false,
        meaning:'',
        meaningsList: [],
        meaningId: '',
          curentAudioId: '',
        editWordDialog: false,
        editedWordId: '',
        editedWord: '',

        editMeaningDialog : false,
        editedMeaning: '',

        confirmDeletionDialog: false,
          customText: '',

        audioTitle: 'No audio available',
        audioDialog: false,
          editAudioDialog: false,
        confirmAudioDeleteDialog: false,
        showProgress: true,

        recordType: 'meaning',
        copyImageDialog: false,
        imageIdToCopy: '',
        currentIdToAddImage: '',

          currentFileIdMarkedForDeletion: '',
          currentRefIdFileIdMarkedForDeletion: '',
          confirmFileDeletionDialog: false,
      }
    },
    components: {
      AddNewMeaning,
      MeaningImage,
      DeleteConfirmationDialog,
      ShowProgressComponent,
      PriorityComponent
    },
    mounted() {
        console.log('current page ::: ShowMeanings.vue');
      this.editedWord = this.word;
      this.showWordMethodInHTML();
      this.getAllMeaningsByWordId('byDate');
    },
    methods : {
        confirmFileDelete(id, refId) {
            this.currentFileIdMarkedForDeletion = id;
            this.currentRefIdFileIdMarkedForDeletion = refId;
            this.customText = 'Are you sure?';
            this.confirmFileDeletionDialog = true;
        },
        dontDeleteFile() {
            this.confirmFileDeletionDialog = false
        },
        fileDeleteOk() {
            this.confirmFileDeletionDialog = false;
            let yourConfig = {
                headers: {
                    Authorization: "Bearer " + this.jwtToken
                },
                params: {
                    'linkId' : this.currentRefIdFileIdMarkedForDeletion,
                    'id' : this.currentFileIdMarkedForDeletion
                }
            };

            axios.delete("http://localhost:8082/genericFileController/delete", yourConfig).then(() => {

                let yourConfig1 = {
                    headers: {
                        Authorization: "Bearer " + this.jwtToken
                    },
                    params: {
                        'linkId' : this.currentRefIdFileIdMarkedForDeletion
                    }
                };

                axios.get("http://localhost:8082/genericFileController/getAllGenericFilesByLinkId", yourConfig1).then(response => {
                    //console.log('response genericFileController ', response.data);
                    this.attachedFileList = [];
                    if(response.data.length != 0) {

                        for (let i = 0; i < response.data.length; i++) {
                            let tmp = {
                                'id': response.data[i].id,
                                'fileName' : response.data[i].fileName
                            };
                            this.attachedFileList.push(tmp);
                        }

                        this.isAttachedFilesAvailable = true;
                    } else {
                        this.isAttachedFilesAvailable = false;
                    }
                }).catch(e => {
                    console.log('ERROR fetching attached files : ', e);
                })

            }).catch(e => {
                console.log('ERROR deleteWord ', e)
            })
        },
        downloadItem(fileId, refId) {
            let yourConfig = {
                headers: {
                    Authorization: "Bearer " + this.jwtToken
                }
            };
            axios.post("http://localhost:8082/genericFileController/downloadFilesById", {
                'linkId' : refId,
                'id': fileId
            }, yourConfig).then(response => {
                const link = document.createElement('a');
                link.href = response.data.fileData;
                link.download = response.data.fileName;
                link.click();
                URL.revokeObjectURL(link.href);
            }).catch(e => {
                console.log('ERROR while downloading file    : ', e)
            })
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
        }).catch(e => {
          console.log('ERROR archive ', e)
        })        
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
                  'refId' : this.meaningId,
                  'audioId' : this.curentAudioId
              }
          };

        axios.get("http://localhost:8082/audioController/deleteAudioById", yourConfig).then(response => {
            for(let i=0; i<this.meaningsList.length;i++) {
                if(this.meaningsList[i].id === this.meaningId) {
                    let yourConfig1 = {
                        headers: {
                            Authorization: "Bearer " + this.jwtToken
                        },
                        params: {
                            'refId' : this.meaningId
                        }
                    };
                    axios.get("http://localhost:8082/audioController/audioByRefIdOrderByDateTime", yourConfig1).then(response => {
                        if(response.data.length != 0) {
                            this.meaningsList[i].audioAvailable = true;
                            this.meaningsList[i].audioList = response.data;
                        } else {
                            this.meaningsList[i].audioAvailable = false;
                        }
                        //this.showProgress = false;
                    }).catch(e => {
                        console.log('ERROR 111 : ', e)
                    });
                    break;
                }
            }
          }).catch(e => {
            console.log('ERRORRORORROOR ', e)
          })
      },
      intro(meaning) {
        return meaning.substring(0, 100);
      },
      showWordMethodInHTML() {
        this.editedWord = this.editedWord.replace(/(\r\n|\n|\r)/gm,"<br>");
        document.getElementById("showWordHere").innerHTML = this.editedWord;
      },
      getAllMeaningsByWordId(token) {

          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              },
              params: {
                  'wordId' : this.wordId,
                  'sortBy': token
              }
          };

        axios.get("http://localhost:8082/meaningController/meaningsByWordId", yourConfig).then(response => {
                  // JSON responses are automatically parsed.
                  this.meaningsList = response.data;
                  this.showProgress = false;

                    for (let index = 0; index < this.meaningsList.length; index++) {

                        let tmpConfig = {
                            headers: {
                                Authorization: "Bearer " + this.jwtToken
                            },
                            params: {
                                'refId': this.meaningsList[index].id
                            }
                        };

                        axios.get("http://localhost:8082/audioController/audioByRefIdOrderByDateTime", tmpConfig).then(response => {
                            if (response.data.length != 0) {
                                for (let i = 0; i < this.meaningsList.length; i++) {
                                    if (this.meaningsList[i].id === response.data[0].refId) {
                                        this.meaningsList[i].audioAvailable = true;
                                        this.meaningsList[i].audioList = response.data;
                                        break;
                                    }
                                }
                            }
                        }).catch(e => {
                            console.log('ERROR 345 : ', e)
                        });

                        let tmpConfig1 = {
                            headers: {
                                Authorization: "Bearer " + this.jwtToken
                            },
                            params: {
                                'wordOrMeaningId': this.meaningsList[index].id
                            }
                        };
                        axios.get("http://localhost:8082/videoController/videosBywordOrMeaningId", tmpConfig1).then(response => {
                            if (response.data.length != 0) {
                                for (let i = 0; i < this.meaningsList.length; i++) {
                                    if (this.meaningsList[i].id === response.data[0].refId) {
                                        this.meaningsList[i].videoAvailable = true;
                                        this.meaningsList[i].videoList = response.data;
                                        break;
                                    }
                                }
                            }
                        }).catch(e => {
                            console.log('ERROR mounted : ', e)
                        });

                        // fetching attachedFiles for this meaning
                        let attachedFileConfig = {
                            headers: {
                                Authorization: "Bearer " + this.jwtToken
                            },
                            params: {
                                'linkId': this.meaningsList[index].id
                            }
                        };
                        axios.get("http://localhost:8082/genericFileController/getAllGenericFilesMetadataByLinkId", attachedFileConfig).then(response => {
                            //console.log(this.meaningsList[index].meaning);
                            //console.log('response.data ', response.data);
                            if (response.data.length != 0) {
                                this.meaningsList[index].isAttachedFilesAvailable = true;
                                this.meaningsList[index].attachedFileList = response.data;
                            }
                        }).catch(e => {
                            console.log('ERROR mounted : ', e)
                        });
                    }

                    // TODO go to bottom - not working
                    //window.scrollTo(0, 0);

                }).catch(e => {
                  console.log('ERROR mounted : ', e);
                  this.showProgress = false;
                })
      },
      saveWord() {
        let newValue = document.getElementById('editWordTextArea').value;

          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              },
              params: {
                  'wordId': this.editedWordId,
                  "newValue": newValue.trim(),
              }
          };
        axios.get("http://localhost:8082/wordController/editWord", yourConfig).then(() => {
                  this.editedWord  = newValue.trim();
                  this.showWordMethodInHTML();
                  this.editWordDialog = false
                                    
                }).catch(e => {
                  console.log('ERROR edit word : ', e)
                })

      },
      editWord(wordId) {
        this.editedWord = this.word;
        this.showWordMethodInHTML();
        this.editedWordId = wordId;
        this.editWordDialog = true

      },
      editMeaning(meaningId, meaning) {
        this.meaningId = meaningId;
        this.editedMeaning = meaning;
        this.editMeaningDialog = true
      },
      saveMeaning() {
        let newVal = document.getElementById('editMeaningTextArea').value;

          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              },
              params: {
                  'meaningId': this.meaningId,
                  "newValue": newVal.trim(),
              }
          };
                axios.get("http://localhost:8082/meaningController/editMeaning", yourConfig).then(() => {
                 
                  this.editMeaningDialog = false;

                  for (let j = 0; j < this.meaningsList.length; j++ ) {
                    if(this.meaningsList[j].id === this.meaningId) {
                      this.meaningsList[j].meaning = newVal.trim();
                    }
                  }

                }).catch(e => {
                  console.log('ERROR edit meaning : ', e)
                })

      },
      addNewMeaning() {
        //console.log('wordId ', this.wordId)
        this.addNewMeaningDialog = true
      },
      addNewMeaningDialogClose() {
        this.addNewMeaningDialog = false;
        //console.log('addNewMeaningDialogClose')
        this.getAllMeaningsByWordId('byDate');
      },
      deleteRecordingButton(meaningId, audioId) {
        this.meaningId = meaningId;
        this.curentAudioId = audioId;
        this.customText = 'Are you Sure?';
        this.confirmAudioDeleteDialog = true;
      },
      editRecordingButton(meaningId, audioId) {
        this.meaningId = meaningId;
        this.curentAudioId = audioId;
        this.audioTitle = 'Click Start to edit voice note';
        document.getElementById('recordingStopButton').style.display = 'none';
        this.editAudioDialog = true;
      },
      recordVoiceMemo(meaningId) {
        this.meaningId = meaningId;
        this.audioDialog = true;
        this.audioTitle = 'Click Start to record your voice note';
        document.getElementById('recordingStopButton').style.display = 'none';
      },
      play(meaningId, audioId) {
        this.meaningId = meaningId;

          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              },
              params: {
                  'refId' : meaningId,
                  'audioId' : audioId
              }
          };
          axios.get("http://localhost:8082/audioController/audioById", yourConfig).then(response => {
              //console.log('this current response ', response.data);
              playRecording(response.data.audio);
          }).catch(e => {
              console.log('ERRORRORORROOR 4  ', e)
          });

      },
      startRecordingMemo() {
        //console.log('startRecordingMemo');
        this.audioTitle = 'Recording voice memo, Click Stop once done';
        document.getElementById('recordButton').style.display = 'none';
        document.getElementById('cancelRecordingButton').style.display = 'none';
        document.getElementById('recordingStopButton').style.display = 'block';
        
        startRecording(this.meaningId);
      },
        startEditingVoiceMemo() {
            document.getElementById('editRecordButton').style.display = 'none';
            document.getElementById('editCancelRecordingButton').style.display = 'none';
            document.getElementById('editRecordingStopButton').style.display = 'block';

            this.audioTitle = 'Editing voice memo, Click Stop once done';
            this.startRecordingMemo();
        },
        stopEditingVoiceMemo() {
            stopEditRecording(this.meaningId, this.curentAudioId, this.meaningsList, this.jwtToken);
            document.getElementById('editRecordButton').style.display = 'block';
            document.getElementById('editCancelRecordingButton').style.display = 'block';
            document.getElementById('editRecordingStopButton').style.display = 'none';
            this.editAudioDialog = false;
        },
      stop() {
        //console.log('stop recording ', this.meaningId);
        stopRecording(this.meaningId, this.meaningsList, this.jwtToken);
        
        this.audioDialog = false;
        document.getElementById('recordButton').style.display = 'block';
        document.getElementById('cancelRecordingButton').style.display = 'block';
        document.getElementById('recordingStopButton').style.display = 'none';
      },
      deleteMeaning(meaningId) {      
        this.meaningId = meaningId;
        this.customText = 'Are you sure? It will remove all data related to this item';
        this.confirmDeletionDialog = true
      },
      dontDelete() {
        this.confirmDeletionDialog = false;
        this.meaningId = ''
      },
      deleteOk() {
        this.confirmDeletionDialog = false;

          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              },
              params: {
                  'meaningId' : this.meaningId
              }
          };
        axios.delete("http://localhost:8082/wordController/deleteMeaning", yourConfig).then((response) => {
          //console.log('deletemeaning response ', response.status)
          if(response.status == 200) {
            for(var i=0; i<this.meaningsList.length;i++) {
              if(this.meaningsList[i].id === this.meaningId) {
                this.meaningsList.splice(i, 1);
                this.meaningId = '';
                break;
              }
            }
          }
        }).catch(e => {
          console.log('ERROR deletemeaning ', e)
        })
        
      },
      close() {
        this.dialog = false;
        this.parentCloseFunc();
      }
    },
    props: [ 'word', 'wordId', 'parentCloseFunc', 'jwtToken' ],
  }
</script>

<style scoped>
  
.responsive {
    width: 100%;
    height: auto;
}

div.ex1 {
  width:40%;
  margin: auto;
}


</style>