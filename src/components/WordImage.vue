<template>
  <div>
      <v-dialog v-model="currentImageDialog" fullscreen hide-overlay transition="dialog-bottom-transition">
          <v-card>
              <v-card-title class="headline">
                  <v-textarea rows="20" row-height="20" outline name="input-7-4" :value=currentWord readonly></v-textarea>
              </v-card-title>
              <v-card-actions>
                  <v-btn color="orange" @click="currentImageDialog = false">Close</v-btn>
                  <v-spacer></v-spacer>
                  <v-btn color="orange" @click="copyImageId">Copy Image Id</v-btn>
                  <v-spacer></v-spacer>
                  <v-btn color="red" @click="confirmDeletion">Delete</v-btn>
              </v-card-actions>
                <v-img v-if="currentImageUrl" :src="currentImageUrl"></v-img>
                <br>
                <v-text-field readonly :id=currentImageId v-model="currentImageId" label="ImageId"></v-text-field>
                <br>
                <v-card-actions>
                  <v-btn color="orange" @click="currentImageDialog = false">Close</v-btn>
                  <v-spacer></v-spacer>
                  <v-btn color="orange" @click="copyImageId">Copy Image Id</v-btn>
                  <v-spacer></v-spacer>
                  <v-btn color="red" @click="confirmDeletion">Delete</v-btn>
                </v-card-actions>
          </v-card>
      </v-dialog>
      <div v-if="showProgress">
          <ShowProgressComponent></ShowProgressComponent>
      </div>
      <div v-if="confirmDeletionDialog">
          <DeleteConfirmationDialog :no=dontDeleteImage :yes=deleteImage />
      </div>
      <div v-if="confirmFileDeletionDialog">
          <DeleteConfirmationDialog :no=dontDeleteFile :yes=fileDeleteOk :customText=customText />
      </div>
      <div v-if="isAttachedFilesAvailable" >
          <br>
          Attached Files: <br>
          <div v-for="item in attachedFileList" :key="item.id">
              <v-btn color="red" @click="confirmFileDelete(item.id, item.linkId)">
                  <v-icon>delete_forever</v-icon>
              </v-btn>
              <v-btn color="green" @click="downloadItem(item.id)">
                  <v-icon>cloud_download</v-icon>
              </v-btn>
              &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {{item.fileName}}
          </div>
          <br>
      </div>
      <table width="100%">
          <tr>
              <td>
                  <div class="file-upload">
                      <div class="image-upload-wrap">
                          <input class="file-upload-input" type='file' @change="onFilePickedGeneric"/>
                          <div class="drag-text">
                              <h2>Click / Drag here to upload file 2</h2>
                          </div>
                      </div>
                  </div>
              </td>
          </tr>
      </table>
      <br>
      <div v-if="images.length != 0">
        <v-carousel>
          <v-carousel-item v-for="(image, i) in images" :key="i" @click="currentImage(image.wordId, image.imageId)" :src="image.src">
          </v-carousel-item>
        </v-carousel>
        <v-carousel>
          <v-carousel-item v-for="(image, j) in 1" :key="j"
                           @click="currentImage(image.wordId, image.imageId)" src="https://i.pinimg.com/564x/4c/9a/ee/4c9aeeb0fd08c9c388a60e2fad75251a.jpg">
          </v-carousel-item>
        </v-carousel>
      </div>
  </div>
</template>

<script>
import axios from 'axios';
import DeleteConfirmationDialog from './DeleteConfirmationDialog';
import ShowProgressComponent from './ShowProgressComponent';

  export default {
    data: () => ({
        showProgress:false,
      images: [],
      currentImageDialog: false,
      currentImageUrl: '',
      currentWordId: '',
      currentImageId: '',
      confirmDeletionDialog: false,
        isAttachedFilesAvailable: false,
        attachedFileList: [],
        confirmFileDeletionDialog: false,
        currentFileId: '',
      allowedVideoFormats: ['video/webm', 'video/mp4'],
      allowedImageFormats: ['image/jpeg', 'image/png'],
    }),
    components: {
      DeleteConfirmationDialog,
        ShowProgressComponent
    },
    methods: {
      copyImageId() {
        /* Get the text field */
          let copyText = document.getElementById(this.currentImageId);

          /* Select the text field */
          copyText.select();
          copyText.setSelectionRange(0, 99999); /*For mobile devices*/

          /* Copy the text inside the text field */
          document.execCommand("copy");

          /* Alert the copied text */
          //alert("Copied the text: " + copyText.value);

          this.currentImageDialog = false;
      },
      deleteCurrentImage() {
        this.currentImageDialog = false;
          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              },
              params: {
                  'wordId' : this.currentWordId,
                  'imageId' : this.currentImageId
              }
          };

        axios.get("http://localhost:8082/imageController/deleteImageByWordIdAndImageId", yourConfig).then(response => {
          this.getAllImagesByWordId();
        }).catch(e => {
          console.log('ERROR mounted : ', e)
        })
      },
      currentImage(wordId, imageId) {
        this.currentImageDialog = true;
        this.currentWordId = wordId;
        this.currentImageId = imageId;

          let yourConfig = {
              headers: {
                  Authorization: "Bearer " + this.jwtToken
              },
              params: {
                  'wordId' : wordId,
                  'imageId' : imageId
              }
          };
        axios.get("http://localhost:8082/imageController/getImageByWordIdAndImageId", yourConfig).then(response => {
          this.currentImageUrl = response.data.image;
        }).catch(e => {
          console.log('ERROR mounted : ', e)
        })
      },
      imageClicked() {
        this.loadMeaningsFn(this.wordId, this.currentWord);
      },
      confirmDeletion() {
        this.confirmDeletionDialog = true
      },
      dontDeleteImage() {
        this.confirmDeletionDialog = false
      },
      deleteImage() {
        this.confirmDeletionDialog = false;
        this.deleteCurrentImage();
      },
      readURL(input) {
          var reader = new FileReader();
          reader.readAsDataURL(input.files[0]);
          //console.log('reader ', reader)
      },
      onFilePickedGeneric(e) {
        this.showProgress = true;
        const uploadedFile = e.target.files[0];
        const uploadedFileType = uploadedFile.type;

        if(this.allowedVideoFormats.includes(uploadedFileType)) {
          this.confirmUploadVideo(e);
        } else if(this.allowedImageFormats.includes(uploadedFileType)) {
          this.uploadImage(e);
        } else {
          this.uploadGenericFile(e);
        }

      },
      confirmUploadVideo(e) {
          // TODO this is redundant method
        const files = e.target.files;
          let fileName = files[0].name.split('.')[0];
          fileName = fileName.split(' - A Cloud Guru')[0];
        if(files[0] != null) {
          const fr = new FileReader ();
            fr.readAsDataURL(files[0]);
            fr.addEventListener('load', () => {

              fr.onloadend = function() {

                  let yourConfig = {
                      headers: {
                          Authorization: "Bearer " + this.jwtToken
                      }
                  };
                  axios.post("http://localhost:8082/videoController/saveOrUpdateVideo", {
                      'refId' : this.wordId,
                      'video' : fr.result,
                      'description': fileName
                  }, yourConfig).then(() => {

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
                              this.responseList[i].videoAvailable = true;
                              this.responseList[i].videoList = response.data;
                            } else {
                              this.responseList[i].videoAvailable = false
                            }
                              this.showProgress = false;
                          }).catch(e => {
                            console.log('ERROR mounted : ', e)
                          });

                        break;
                      }
                    }


                  }).catch(e => {
                    console.log('notes upload ERROR : ', e)
                  })
              }.bind(this)

            })
        }
      },
        downloadItem(fileId) {
          // TODO duplicate method in MeaningImage.vue
            let yourConfig = {
                headers: {
                    Authorization: "Bearer " + this.jwtToken
                }
            };
            axios.post("http://localhost:8082/genericFileController/downloadFilesById", {
                'linkId' : this.wordId,
                'id': fileId
            }, yourConfig).then(response => {
                const link = document.createElement('a');
                link.href = response.data.fileData;
                link.download = response.data.fileName;
                link.click();
                URL.revokeObjectURL(link.href);
            }).catch(e => {
                console.log('ERROR while downloading file    : ', e);
            })
        },
        dontDeleteFile() {
            // TODO duplicate method in MeaningImage.vue
            this.confirmFileDeletionDialog = false
        },
        fileDeleteOk() {
            // TODO duplicate method in MeaningImage.vue
            this.confirmFileDeletionDialog = false;
            let yourConfig = {
                headers: {
                    Authorization: "Bearer " + this.jwtToken
                },
                params: {
                    'linkId' : this.wordId,
                    'id' : this.currentFileId
                }
            };

            axios.delete("http://localhost:8082/genericFileController/delete", yourConfig).then(() => {

                let yourConfig1 = {
                    headers: {
                        Authorization: "Bearer " + this.jwtToken
                    },
                    params: {
                        'linkId' : this.wordId
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
        confirmFileDelete(id) {
            // TODO duplicate method in MeaningImage.vue
            this.currentFileId = id;
            this.customText = 'Are you sure?';
            this.confirmFileDeletionDialog = true;
        },
        uploadGenericFile(e) {
            // TODO duplicate method in MeaningImage.vue
            const files = e.target.files;
            //console.log('ffffffff ', files[0]);
            const fr = new FileReader ();
            fr.readAsDataURL(files[0]);
            fr.addEventListener('load', () => {

                fr.onloadend = function() {
                    let yourConfig = {
                        headers: {
                            Authorization: "Bearer " + this.jwtToken
                        }
                    };

                    axios.post("http://localhost:8082/genericFileController/save", {
                        'linkId' : this.wordId,
                        'fileData' : fr.result,
                        'fileName' : files[0].name,
                        'type': files[0].type
                    }, yourConfig).then((response) => {
                        console.log('wordImage response.data ', response.data);
                        let obj = {
                            'id': response.data.id,
                            'linkId': response.data.linkId,
                            'fileName' : response.data.fileName,
                            'type' : response.data.type,
                            'dateTime' : response.data.dateTime
                        };
                        this.attachedFileList.push(obj);
                        this.isAttachedFilesAvailable = true;
                    }).catch(e => {
                        console.log('notes upload ERROR : ', e)
                    });
                    this.showProgress = false;
                }.bind(this)

            })

        },
      uploadImage (e) {
          //console.log('onFilePicked ', e);
// TODO duplicate method in MeaningImage.vue
          const files = e.target.files;
          if(files[0] != undefined) {
            this.imageName = files[0].name;
            if(this.imageName.lastIndexOf('.') <= 0) {
              return
            }

            const fr = new FileReader ();
            fr.readAsDataURL(files[0]);
            fr.addEventListener('load', () => {
              this.imageFile = files[0]; // this is an image file that can be sent to server...

              //console.log('this.imageFile ', this.imageFile)

              var myfr = new FileReader();
              myfr.readAsDataURL(this.imageFile);
              myfr.onloadend = function() {
                  let yourConfig = {
                      headers: {
                          Authorization: "Bearer " + this.jwtToken
                      }
                  };
                  axios.post("http://localhost:8082/imageController/saveOrUpdateImage", {
                      'wordId' : this.wordId,
                      'image' : myfr.result
                  }, yourConfig).then(() => {
                    //var base64_decode = new Buffer(response.data, 'base64');
                    //console.log('base64_decode ', base64_decode);
                    this.getAllImagesByWordId();
                  }).catch(e => {
                    console.log('Image upload ERROR : ', e)
                  });
                  this.showProgress = false;
              }.bind(this)
            })
          } else {
            this.imageName = '';
            this.imageFile = ''
          }
        },
        getAllImagesByWordId() {
// TODO duplicate method in MeaningImage.vue
            let yourConfig = {
                headers: {
                    Authorization: "Bearer " + this.jwtToken
                },
                params: {
                    'wordId' : this.wordId
                }
            };

          axios.get("http://localhost:8082/imageController/getAllImagesByWordId", yourConfig).then(response => {
                    this.images = [];
                    if(response.data != null && response.data != undefined && response.data != '') {

                      for(let i=0; i<response.data.length;i++) {
                        let myJson = {
                          "src" : response.data[i].image,
                          "wordId": response.data[i].wordId,
                          "imageId": response.data[i].imageId
                        };
                        this.images.push(myJson);
                      }
                    } else {
                      /*
                      let myJson = {
                          "src" : 'aws_certification_path.png',
                          "wordId": '',
                          "imageId": ''
                        }
                        this.images.push(myJson);
                        */
                    }
                  }).catch(e => {
                    console.log('ERROR mounted : ', e)
                  })
        },
        getAllGenericFiles() {
            // TODO this is redundant method
            let yourConfig = {
                headers: {
                    Authorization: "Bearer " + this.jwtToken
                },
                params: {
                    'linkId' : this.wordId
                }
            };
            axios.get("http://localhost:8082/genericFileController/getAllGenericFilesByLinkId", yourConfig).then(response => {
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
                }
            }).catch(e => {
                console.log('ERROR fetching attached files : ', e);
            })
        }
    },
    mounted() {
      this.getAllImagesByWordId();
        this.getAllGenericFiles();
    },
    props: ['responseList', 'wordId', 'loadMeaningsFn', 'currentWord', 'jwtToken']
  }
</script>

<style scoped>

  .responsive {
      width: 100%;
      height: auto;
  }

  body {
  font-family: sans-serif;
  background-color: #eeeeee;
}

.file-upload {
  background-color: black;
  width: 100%;
  margin: 0 auto;
  padding: 20px;
}

.file-upload-btn {
  width: 100%;
  margin: 0;
  color: #fff;
  background: #1FB264;
  border: none;
  padding: 10px;
  border-radius: 4px;
  border-bottom: 4px solid #15824B;
  transition: all .2s ease;
  outline: none;
  text-transform: uppercase;
  font-weight: 700;
}

.file-upload-btn:hover {
  background: #1AA059;
  color: #ffffff;
  transition: all .2s ease;
  cursor: pointer;
}

.file-upload-btn:active {
  border: 0;
  transition: all .2s ease;
}

.file-upload-content {
  display: none;
  text-align: center;
}

.file-upload-input {
  position: absolute;
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  outline: none;
  opacity: 0;
  cursor: pointer;
}

.image-upload-wrap {
  margin-top: 20px;
  border: 4px dashed #1FB264;
  position: relative;
}

.image-dropping,
.image-upload-wrap:hover {
  background-color: #1FB264;
  border: 4px dashed #ffffff;
}

.image-title-wrap {
  padding: 0 15px 15px 15px;
  color: #222;
}

.drag-text {
  text-align: center;
}

.drag-text h3 {
  font-weight: 100;
  text-transform: uppercase;
  color: #15824B;
  padding: 60px 0;
}

.file-upload-image {
  max-height: 200px;
  max-width: 200px;
  margin: auto;
  padding: 20px;
}

.remove-image {
  width: 200px;
  margin: 0;
  color: #fff;
  background: #cd4535;
  border: none;
  padding: 10px;
  border-radius: 4px;
  border-bottom: 4px solid #b02818;
  transition: all .2s ease;
  outline: none;
  text-transform: uppercase;
  font-weight: 700;
}

.remove-image:hover {
  background: #c13b2a;
  color: #ffffff;
  transition: all .2s ease;
  cursor: pointer;
}

.remove-image:active {
  border: 0;
  transition: all .2s ease;
}

</style>
