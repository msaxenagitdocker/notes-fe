<template>
  <v-app id="inspire" dark>
    <v-snackbar top v-model="snackbar" :multi-line="multiLine">
      {{ snackbarText }}
        <v-btn color="red" text @click="snackbar = false">
          Close
        </v-btn>
    </v-snackbar>

    <div v-if="dialog">
      <AddNewWord :wordType=selectedItem :parentCloseFunc="close" :jwtToken=jwtToken />
    </div>

    <div v-if="categoryDialog">
      <EditCollections :menuItems=menuItems :parentCloseFunc="closeCategoryDialog"/>
    </div>

    <div v-if="showProgress">
        <ShowProgressComponent></ShowProgressComponent>
      </div>


    <v-navigation-drawer v-model="drawer" fixed clipped app>

    <v-list >
        <v-list-tile @click.native="addNewWord">
          <v-list-tile-action>
            <v-icon>add</v-icon>
          </v-list-tile-action>
          <v-list-tile-content>
            <v-list-tile-title>
              Add New Word
            </v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>



    </v-navigation-drawer>
    <v-toolbar color="blue" app fixed clipped-left>
      <v-toolbar-side-icon @click="drawer = !drawer"></v-toolbar-side-icon>
      <span class="title ml-3 mr-5">Svenska Learning</span>
      <v-spacer></v-spacer>
      <v-icon @click="category()">addition</v-icon>
      <v-icon @click="search('search')">refresh</v-icon>
      <v-spacer></v-spacer>
      <v-text-field @keyup.enter="search('search')" v-model="query"
        solo-inverted flat hide-details label="Search" prepend-inner-icon="search"></v-text-field>



  <!-- drop down code start -->
      <v-layout>
        <v-flex >
          <v-select dense
          outlined v-model="selectedItem" @change="search('search')"
            :items="menuItems" label="Select type" ></v-select>
        </v-flex>
    </v-layout>

    <!-- drop down code end -->
<v-spacer></v-spacer>

    <v-switch v-model="searchInMeanings" :label="'CHECK Search In Meanings'"
    @change="search('search')"></v-switch>

<v-spacer></v-spacer>

<v-switch v-model="sortingOrder" :label="'Sorting Order'"
    @change="sortingOrderSearch"></v-switch>

<v-spacer></v-spacer>
<v-switch v-model="priorityOrder" :label="'Priority'"
    @change="prioritySearch"></v-switch>

<v-spacer></v-spacer>
<v-icon v-if="priorityOrder">arrow_downward</v-icon>
<v-icon v-else>arrow_upward</v-icon>
<v-spacer></v-spacer>

      <v-btn color="orange" @click="addNewWord">Add New Word</v-btn>
      <v-spacer></v-spacer>
      Displayed : {{(totalWordsDisplayed - 10) < 0 ? 0 : (totalWordsDisplayed - 10)}} ... {{totalWordsDisplayed}}
      <v-spacer></v-spacer>
      Total : {{totalWordsAvailable}}
    </v-toolbar>

<div width="100%">
        <v-btn color="orange" @click="search('prev')">Previous Page</v-btn>
        <v-btn color="orange" @click="search('next')">Next Page</v-btn>
</div>

    <v-content>
      <Card :jwtToken=jwtToken :menuItems=menuItems :responseList=responseList :parentCloseFunc="getWordCount" :selectedWordType="selectedItem" :refreshSearchFn="refreshSearch" />
    </v-content>

    <br>
    <div width="100%">
        <v-btn color="orange" @click="search('prev')">Previous Page</v-btn>
        <v-btn color="orange" @click="search('next')">Next Page</v-btn>
    </div>

  </v-app>
</template>

<script>
import axios from 'axios';
import Card from './Card'
import AddNewWord from './AddNewWord'
import ShowProgressComponent from './ShowProgressComponent';
import EditCollections from './EditCollections.vue';

  export default {
    components: {
      Card,
      AddNewWord,
      ShowProgressComponent,
      EditCollections
    },
    data: () => ({
      // youtube video upload - start
      /*
      title: '',
      description: '',
      tags: [],
      */
      // youtube video upload - end

      sortingOrder: true,
      priorityOrder: true,
      searchInMeanings: false,
      totalWordsDisplayed: 0,
      totalWordsAvailable: 0,
      selectedItem:'',
      menuItems: ['Notes', 'Architect', 'Ongoing', 'Kafka', 'Health', 'AWS', 'Azure', 'Terraform', 'Mark', 'Quotes', 'TODO', 'Svenska', 'Docker & Kubernetes', 'Driving', 'Stocks', 'ALL'],
      categoryDialog: false,
      query: '',
      responseList: [],
      dialog: false,
      startPage: 0,
      drawer: false,
      items: [
        { icon: 'add', text: 'Add New Word' },
        { divider: true },
        { icon: 'touch_app', text: 'Reminders' },
        { icon: 'archive', text: 'Archive' },
        { icon: 'delete', text: 'Trash' },
        { divider: true },
        { icon: 'settings', text: 'Settings' },
        { icon: 'help', text: 'Help' }
      ],
      items2: [
        { picture: 28, text: 'Joseph' },
        { picture: 38, text: 'Apple' }
      ],

      showProgress: false,
      multiLine: true,
      snackbar: false,
      snackbarText: 'Nothing Found',
      order: '',
      isPrioritySearch : false,
        jwtToken: ''

    }),
    mounted() {

      // youtube video upload - start
/*
      const client_id = "140810889562-png4krtv5h4a38olcepd35tara8r4sn9.apps.googleusercontent.com"
      const project_id = "language-355218"
      const auth_uri = "https://accounts.google.com/o/oauth2/auth"
      const token_uri = "https://oauth2.googleapis.com/token"
      const auth_provider_x509_cert_url = "https://www.googleapis.com/oauth2/v1/certs"
      const client_secret = "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"
      const redirect_uris = ["http://localhost:8080/google/callback"]
      const javascript_origins = ["http://localhost:8080"]

      const oAuth2Client = new google.auth.OAuth2(
          client_id,
          client_secret,
          redirect_uris[0]
      );
      const SCOPES = "https://www.googleapis.com/auth/youtube.upload https://www.googleapis.com/auth/userinfo.profile";
      const scopes1 = [
        'https://www.googleapis.com/auth/blogger',
        'https://www.googleapis.com/auth/calendar'
      ];
      console.log('oAuth2Client ', oAuth2Client)

 */
// youtube video upload - end



        let config = {
            headers: {
                'Content-Type': 'application/json'
            }
        };

        axios.post("http://localhost:8082/securityController/authentication", {
            "userName": "manish",
            "password": "saxena"
        }, config).then(response => {
            this.jwtToken = response.data.jwtToken;
            //console.log('jwtToken ', this.jwtToken);

            let yourConfig = {
                headers: {
                    Authorization: "Bearer " + this.jwtToken
                },
                params: {
                    'query' : this.query.trim(),
                    'type': this.selectedItem
                }
            };

            axios.get("http://localhost:8082/wordController/wordCount", yourConfig).then(response => {
                //console.log('response.data ', response.data);
                this.totalWordsAvailable = response.data
            }).catch(e => {
                console.log('ERROR search ------------------------: ', e)
            })

        }).catch(e => {
            console.log('ERROR while downloading file    : ', e)
        });


    },
    created() {
      /*
      let config = {
        headers: {
          "Api-Key": "cf6569a2c9d14c91aa377a4de814c82b",
          "Api-Timestamp": new Date().getTime(),
          "Api-Content-Hash": "",
          "Api-Signature": ""
        }
      }

      //setInterval(() => this.getCurrentTime(), 5000);

      console.log('config ', config);

        axios.get("https://api.bittrex.com/v3/markets", config)
        .then(response => {
          console.log('BITTREX response ::: ', response);
        }).catch(e => {
          console.log('BITTREX ERROR : ', e)
        })
*/

    },
    methods : {
      closeCategoryDialog() {
        this.categoryDialog = false;
      },
      category() {
        this.categoryDialog = true;
        console.log(this.menuItems);
      },
      getCurrentTime() {
          let d = new Date();
          console.log(d.toString().substring(0, d.toString().split('GMT')[0].lastIndexOf(":")).trim());
          //console.log("Current Time is ::: " + d.getHours() + ":" + d.getMinutes()
          //   + ":" + d.getDay()  + ":" + d.getDate());

          // fetch all TODO time from DB and match with this time
          // if matches then show in dialog all of them

      },
      sortingOrderSearch() {

        if(this.sortingOrder) {
          this.order = "DESC"
        } else {
          this.order = "ASC"
        }
        this.isPrioritySearch = false;
        this.search('search');

      },
      prioritySearch() {

        if(this.priorityOrder) {
          this.order = "DESC"
        } else {
          this.order = "ASC"
        }
        this.isPrioritySearch = true;
        this.search('search');

      },
      getWordCount() {
        this.totalWordsAvailable -= 1;
        this.totalWordsDisplayed -= 1
      },

      refreshSearch(token) {
        if(token != '' && token != null && token != undefined) {
          this.isPrioritySearch = true;
        }
        this.search('search');
      },

      search(token) {
        this.drawer = false;

        if(this.query == '' || this.query.trim() == '') {
          this.query = '%'
        }

        if(this.selectedItem == undefined || this.selectedItem == null || this.selectedItem == '') {
          this.selectedItem = 'ALL'
        }

        if(token === 'search') {
          this.startPage = 0;
          this.totalWordsDisplayed = 0
        } else if(token === 'prev') {
          this.startPage -= 10;
          if(this.startPage < 0) {
            this.startPage = 0
          }
          this.totalWordsDisplayed -= 10;
        } else if(token === 'next') {
          this.startPage += 10;
          this.totalWordsDisplayed += 10;
        }

        this.showProgress = true;


              let yourConfig = {
                  headers: {
                      Authorization: "Bearer " + this.jwtToken
                  },
                  params: {
                      'query' : this.query.trim(),
                      'startCounter' : this.startPage,
                      'type': this.selectedItem,
                      'isMeaningSearch': this.searchInMeanings,
                      'order': this.order,
                      'isPrioritySearch': this.isPrioritySearch
                  }
              };

        axios.get("http://localhost:8082/wordController/wordByQuery", yourConfig).then(response => {
            this.showProgress = false;

          this.responseList = [];
          if(this.totalWordsDisplayed == 0) {
            this.totalWordsDisplayed = response.data.length
          }

          this.totalWordsAvailable = 0;

          if(response.data.length == 0) {
            this.snackbar = true;
            return
          }

          this.responseList = response.data;


            yourConfig = {
                headers: {
                    Authorization: "Bearer " + this.jwtToken
                },
                params: {
                    'query' : this.query.trim(),
                    'type': this.selectedItem
                }
            };
            axios.get("http://localhost:8082/wordController/wordCount", yourConfig).then(response => {
              //console.log('response.data ', response.data)
              this.totalWordsAvailable = response.data;

              if(this.totalWordsAvailable < this.totalWordsDisplayed) {
                this.totalWordsAvailable = this.totalWordsDisplayed;
              }

            }).catch(e => {
              this.showProgress = false;
              console.log('ERROR search : ', e)
            });


            //console.log('SEARCH response ::: ', response.data);

            for (let index = 0; index < this.responseList.length; index++) {


                yourConfig = {
                    headers: {
                        Authorization: "Bearer " + this.jwtToken
                    },
                    params: {
                        'refId': this.responseList[index].id
                    }
                };

                axios.get("http://localhost:8082/audioController/audioByRefIdOrderByDateTime", yourConfig).then(response => {
                    if (response.data.length != 0) {
                        for (let i = 0; i < this.responseList.length; i++) {
                            if (this.responseList[i].id === response.data[0].refId) {
                                this.responseList[i].audioAvailable = true;
                                this.responseList[i].audioList = response.data;
                                break;
                            }
                        }
                    }
                }).catch(e => {
                    console.log('ERROR 345 : ', e)
                });


                yourConfig = {
                    headers: {
                        Authorization: "Bearer " + this.jwtToken
                    },
                    params: {
                        'wordOrMeaningId' : this.responseList[index].id
                    }
                };
                axios.get("http://localhost:8082/videoController/videosBywordOrMeaningId", yourConfig).then(response => {
                    if (response.data.length != 0) {
                        for (let i = 0; i < this.responseList.length; i++) {
                            if (this.responseList[i].id === response.data[0].refId) {
                                this.responseList[i].videoAvailable = true;
                                this.responseList[i].videoList = response.data;
                                break;
                            }
                        }
                    }
                }).catch(e => {
                    console.log('ERROR mounted : ', e)
                });


                yourConfig = {
                    headers: {
                        Authorization: "Bearer " + this.jwtToken
                    },
                    params: {
                        'wordId' : this.responseList[index].id
                    }
                };
                axios.get("http://localhost:8082/meaningController/onlyMeaningsByWordId", yourConfig).then(response => {
                    if (response.data.length != 0) {
                        for (let i = 0; i < this.responseList.length; i++) {
                            if (this.responseList[i].id === response.data[0].word) {
                                this.responseList[i].meaningAvailable = true;
                                this.responseList[i].meaningCount = response.data.length;
                                break;
                            }
                        }
                    }
                }).catch(e => {
                    console.log('ERROR mounted : ', e)
                });


                yourConfig = {
                    headers: {
                        Authorization: "Bearer " + this.jwtToken
                    },
                    params: {
                        'linkId' : this.responseList[index].id
                    }
                };
                axios.get("http://localhost:8082/genericFileController/getAllGenericFilesMetadataByLinkId", yourConfig).then(response => {
                    console.log('response.data ', response.data);
                    // TODO - check this call, I am trying to bring all generic file metadata in the first search call only
                    /*if (response.data.length != 0) {
                        for (let i = 0; i < this.responseList.length; i++) {
                            if (this.responseList[i].id === response.data[0].word) {
                                this.responseList[i].meaningAvailable = true;
                                this.responseList[i].meaningCount = response.data.length;
                                break;
                            }
                        }
                    }*/
                }).catch(e => {
                    console.log('ERROR mounted : ', e);
                });
            }

        }).catch(e => {
          this.showProgress = false;
          console.log('ERROR search : ', e)
        })

      },
      addNewWord() {
        this.dialog = true;
      },
      close() {
        this.dialog = false;
        this.drawer = false;
        this.search();
      }
    },
    props: {
      source: String
    }
  }
</script>
