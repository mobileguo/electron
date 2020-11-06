<template>
  <v-app>
    <v-navigation-drawer v-model="drawer" app>
      <v-list dense>
        <v-list-item link>
          <v-list-item-action>
            <v-icon>mdi-home</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>问题反馈:xxx@xxxx.com</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar app color="indigo" dark>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>图片数据标识工具</v-toolbar-title>
    </v-app-bar>

    <v-main style="text-align: center">
      <v-file-input
        style="margin:10px"
        lable="Image input"
        v-model="imageFile"
        filled
        prepend-icon="mdi-message-image-outline"
      ></v-file-input>
      <v-container fluid>
        <HelloWorld v-if="imageFile != null" key="1" :msg="imageFile" />
      </v-container>
    </v-main>
    <v-footer color="indigo" app>
      <span class="white--text">&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld'

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  props: {
    source: String
  },
  data: () => ({
    //
    drawer: null,
    imageFile: null
  }),
  mounted () {
    // console.log(this.$data.imageFile)
    if (this.$data.imageFile != null){
      this.$data.videoHelp = false
    }
  },
  methods: {
    playhelp (){
      this.$data.videoHelp = true
    },
    closevideo (){
      this.$data.videoHelp = false
    }
  },
}
</script>
