<template>
  
        <v-dialog
          v-model="dialog"
          max-width="500px"
        >

<v-card
    class="mx-auto"
    max-width="500"
  >
    <v-list shaped>
      <v-list-item-group
        v-model="model"
        multiple
      >
        <template v-for="(item, i) in menuItems">
          <v-divider
            v-if="!item"
            :key="`divider-${i}`"
          ></v-divider>

          <v-list-item
            v-else
            :key="`item-${i}`"
            :value="item"
            active-class="deep-purple--text text--accent-4"
          >
            <template v-slot:default="{ active }">
              <v-list-item-content>
                <v-list-item-title v-text="item"></v-list-item-title>
              </v-list-item-content>

              <v-list-item-action>
                <v-checkbox
                  :input-value="active"
                  color="deep-purple accent-4"
                ></v-checkbox>
              </v-list-item-action>
            </template>
          </v-list-item>
        </template>
      </v-list-item-group>
    </v-list>
  </v-card>

 <v-btn color="orange" @click="close">CLOSE</v-btn>

        </v-dialog>
  
</template>


<script>
  export default {
    data: () => ({
      dialog: true,
      model: [],
    }),

    props: [ 'menuItems', 'closeCategoryDialog' ],

    created () {
    },

    methods: {

      close() {
        this.dialog = false;
        this.closeCategoryDialog();
      }
    },
  }
</script>