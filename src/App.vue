<template>
  <v-app>
    <v-app-bar app color="secondary">
      <v-spacer></v-spacer>

      <v-btn href="https://github.com/vuetifyjs/vuetify/releases/latest" target="_blank" text>
        <span class="mr-2">Latest Release</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-content>
      <v-file-input @change="handleVideoUpload" />

      <v-btn v-if="showBtn" @click="sendToServer">send to server</v-btn>
      <v-btn v-if="showBtn" @click="getFromServer">get from server</v-btn>
      <videoPlayer :options="lrPlayerOptions" v-if="show" />
      <videoPlayer :options="hrPlayerOptions" v-if="show" />
    </v-content>
  </v-app>
</template>

<script>
import "video.js/dist/video-js.css";
import { videoPlayer } from "vue-video-player";

import axios from "axios";

export default {
  name: "App",

  components: {
    videoPlayer
  },

  data: () => ({
    file: 0,
    show: false,
    server_url: "http://127.0.0.1:5000",
    showBtn: false,
    lrPlayerOptions: {
      sources: []
    },
    hrPlayerOptions: {
      sources: []
    }
  }),
  computed: {
    playerOptions: function() {
      return {
        //sources: [{type: "video/mp4", src: this.file}]
      };
    }
  },
  methods: {
    handleVideoUpload(e) {
      this.file = e;
      this.showBtn = true;
    },
    sendToServer() {
      let form_data = new FormData();
      form_data.append("file", this.file);
      axios.post(`${this.server_url}/video`, form_data, {
        headers: {
          "Content-Type": `multipart/form-data; boundary=${form_data._boundary}`
        }
      });
    },
    getFromServer() {
      axios.get(`${this.server_url}/hr_video`).then(res => {
        if (res.data != "NO") {
          let links = res.data;
          console.log(links);
          let hr_link = links["hr_video_route"];
          let lr_link = links["lr_video_route"];
          this.lrPlayerOptions.sources.push({
            type: "video/mp4",
            src: `${this.server_url}/${lr_link}`,
            autoplay: "muted"
          });
          this.hrPlayerOptions.sources.push({
            type: "video/mp4",
            src: `${this.server_url}/${hr_link}`,
            autoplay: "muted"
          });

          this.show = true;
        }
      });
    }
  }
};
</script>
