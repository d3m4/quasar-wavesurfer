<template lang="pug">
  <q-page class>
    <div class="audio-container">
      <div class="row q-ma-md">
        <div class="col-12">
          <div id="waveform"></div>
        </div>
      </div>
    </div>
    <div class="controls row flex flex-center fixed-bottom q-pb-md q-pt-md shadow-10">
      <div class>
        <q-btn color="primary" flat round icon="fast_rewind" size="xl" @click="wavesurfer.skipBackward(1)" />
        <q-circular-progress v-if="isLoading" size="72px" indeterminate color="primary" />
        <q-btn v-if="isPlaying && !isLoading" color="primary" round icon="pause" size="xl" @click="wavesurfer.playPause()" />
        <q-btn v-if="!isPlaying && !isLoading" color="primary" round icon="play_arrow" size="xl" @click="wavesurfer.playPause()" />
        <q-btn color="primary" flat round icon="fast_forward" size="xl" @click="wavesurfer.skipForward(1)" />
      </div>
    </div>
  </q-page>
</template>

<script>
import WaveSurfer from "wavesurfer.js";
import EventBus from "../services/event-bus";

export default {
  name: 'PageIndex',
  data: () => ({
    wavesurfer: null,
    isLoading: false,
  }),
  async mounted() {
    if (!this.wavesurfer) this.createWaveSurfer();

    EventBus.$on("fileChosen", file => {
      this.loadFile(file);
    })
  },
  computed: {
    isPlaying() {
      if(!this.wavesurfer) return false;

      return this.wavesurfer.isPlaying();ß
    }
  },
  methods: {
    createWaveSurfer() {
      this.wavesurfer = WaveSurfer.create({
        container: "#waveform",
        hideScrollbar: true,
        waveColor: "white",
        progressColor: "hsla(200,100%, 30%, 0.5)",
        cursorColor: "#fff",
        barWidth: 3
      });

      this.wavesurfer.load(
        "https://ia902606.us.archive.org/35/items/shortpoetry_047_librivox/song_cjrg_teasdale_64kb.mp3"
      );

      this.wavesurfer.on("error", err => {
        console.error(err);
        this.isLoading = false;
        this.$q.notify({ message: err });
      });

      this.wavesurfer.on("loading", () => {
        this.isLoading = true;
      });

      this.wavesurfer.on("ready", () => {
        this.isLoading = false;
      })
    },
    loadFile(file) {
      if(file.target.files.length == 0) return;

      this.wavesurfer.loadBlob(file.target.files[0]);
    }
  }
}
</script>

<style>
  .controls{
    background-color: white;
  }

  .audio-container {
    position: absolute;
    top:0;
    left:0;
    height: 100%;
    width: 100%;
    background: linear-gradient(rgba(0,0,0, 0.3), rgba(0,0,0,0.3)), url("../assets/audio.jpg") no-repeat center;
    background-size: cover;
  }
</style>
