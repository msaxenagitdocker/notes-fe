<template>
  <v-layout row justify-center>
  <v-dialog v-model=show persistent max-width="400">
      <v-card>
        <v-card-title class="headline">{{customText}}</v-card-title>
        <br>
        <v-card-actions>
          <v-btn color="red" @click="yesFunc">ok</v-btn>
          <v-spacer></v-spacer>
          <v-btn color="green" @click="noFunc">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    </v-layout>
</template>

<script>
  export default {
    data: () => ({
      show: true
    }),
    methods: {
      noFunc() {
        this.no()
      },
      yesFunc() {
        this.yes();
      }
    },
    props: ['no', 'yes', 'customText']
  }
</script>
