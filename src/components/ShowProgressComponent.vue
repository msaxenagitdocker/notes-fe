<template>
  <div class="text-center">
    <v-dialog
      v-model="dialog"
      hide-overlay
      persistent
      width="300"
    >
      <v-card dark>
        <v-card-text>
          Please stand by
          <v-progress-linear
            indeterminate
            color="white"
          
          ></v-progress-linear>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
  export default {
    data: () => ({
      dialog: true,
    }),
    components: {    
    },
    methods: {
    },
    mounted() {
    },
  }
</script>

<style scoped>


</style>