<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-card
          ><v-card-title>Upload</v-card-title>
          <v-file-input
            accept="image/*"
            append-outer-icon="mdi-send"
            @click:append-outer="upload"
            v-model="uploadFile"
          ></v-file-input>
        </v-card>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="6" v-for="(item, index) in items" :key="index">
        <v-dialog width="unset">
          <template v-slot:activator="{ on, attrs }">
            <v-card>
              <v-img
                contain
                aspect-ratio="1.7"
                :src="item.url"
                max-height="400"
                v-bind="attrs"
                v-on="on"
              ></v-img>
              <v-card-text>{{ item.name }}</v-card-text>
            </v-card>
          </template>
          <template v-slot:default="dialog">
            <v-card>
              <amplify-s3-image
                width="100%"
                class="d-flex justify-center"
                :img-key="item.id"
                level="private"
                @click="dialog.value = false"
              />
            </v-card>
          </template>
        </v-dialog>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { API, graphqlOperation, Storage } from 'aws-amplify'
import { listItems } from '@/graphql/queries'
import { createItem } from '@/graphql/mutations'

export default {
  data: function () {
    return {
      uploadFile: null,
      file: null,
      items: [],
    }
  },
  created: async function () {
    await this.listItems()
  },
  methods: {
    upload: async function () {
      // create new graphql record
      const createdItem = await API.graphql(
        graphqlOperation(createItem, {
          input: {
            name: this.uploadFile.name,
          },
        }),
      )

      // upload to s3
      try {
        await Storage.put(createdItem.data.createItem.id, this.uploadFile, {
          level: 'private',
          contentType: this.uploadFile.type,
        })
        await this.listItems()
      } catch (err) {
        console.error('Error uploading file: ', err)
      }

      this.uploadFile = null
    },
    listItems: async function () {
      const listedItems = await API.graphql(graphqlOperation(listItems))
      this.items = await this.urlAdd(listedItems.data.listItems.items)
    },
    urlAdd: async function (items) {
      const result = await Promise.all(
        items.map(async (item) => {
          item.url = await Storage.get(item.id, {
            level: 'private',
          })
          return item
        }),
      )

      return result
    },
  },
}
</script>

<style scoped>
amplify-s3-image {
  --width: 90vw;
}
</style>
