<template>
  <div id="app">
    <div class="text-center">
      <h1 class="title">slowed + reverb</h1>
    </div>
    <div class="paper container container-sm margin-top-large">
    <div v-if="error"  class="alert alert-danger dismissible">
      {{error}}
      <label @click="error = null" class="btn-close" for="alert-5">X</label>
    </div>
      <div class="row flex-center margin-bottom-large">
        <input type="file" @change="start">
      </div>
      <div v-if="fileloading" class="progress margin-bottom">
        <div :class="'bar striped success w-'+fileloading"></div>
      </div>
      <div class="form-group">
        <label for="percentage">playback rate</label>
        <input class="input-block" type="range" step="0.01" min="0.5" max="1" @mouseup="datachanged" v-model="playback" oninput="output.value = (this.value*100) + '%';">
        <output id="output" for="percentage">{{Math.round(playback*100)}}%</output>
      </div>
      <div class="form-group">
        <label for="percentage">reverb wet</label>
        <input class="input-block" type="range" step="0.01" min="0" max="1" @mouseup="datachanged" v-model="wet" oninput="output.value = (this.value*100) + '%';">
        <output id="output" for="percentage">{{Math.round(wet*100)}}%</output>
      </div>
      <button class="btn-block" @click="play">{{ isPlaying ? 'PAUSE' : 'PLAY' }}</button>
      <button class="btn-block margin-top" @click="replay">REPLAY</button>
      <button class="btn-block btn-secondary-outline margin-top" @click="render">DOWNLOAD</button>
      <a ref="rendered" style="display: none;"></a>
      <div class="row flex-center margin-top">
        <h5>made with 🖤 by <a href="https://www.instagram.com/ad0uz/" target="_blank">adouz</a></h5>
      </div>
    </div>
  </div>
</template>

<script>
import * as Tone from 'tone'
import * as toWav from 'audiobuffer-to-wav'
import '../node_modules/papercss/dist/paper.css'
export default {
  title: '𝓼𝓵𝓸𝔀𝓮𝓭 + 𝓻𝓮𝓿𝓮𝓻𝓫 Generator',
  name: 'App',
  components: {
  },
  data (){
    return {
      buffer: null,
      fileurl: null,
      player: null,
      offset:0,
      startedAt:0,
      playback: 0.8,
      wet: 0.6,
      decay: 2,
      preDelay: 0.0,
      chunks: [],
      renderTime: 60,
      renderFileName: 'none.wav',
      fileloading: false,
      error: null,
      isPlaying: false
    }
  },
  mounted: function (){
  },
  methods:{
    datachanged(){
      if (!this.fileurl) return this.error = 'upload audio first';
      this.pause();
      this.play();
    },
    replay(){
      if (!this.fileurl) return this.error = 'upload audio first';
      if (this.isPlaying) this.pause();
      this.offset = 0;
      this.startedAt = 0;
      this.play();
    },
    start(e){
      if (e.target.files){
        this.fileloading = "20";
        const file = e.target.files[0];
        this.fileurl =  URL.createObjectURL(file);
        this.renderFileName = file.name.split('.')[0] + " 𝓼𝓵𝓸𝔀𝓮𝓭 + 𝓻𝓮𝓿𝓮𝓻𝓫";
      }
      if (this.player && this.player.state === "started") this.stop();
      this.offset = 0;
      this.startedAt = 0;
      const audio = new Audio();
      audio.src = this.fileurl;
      this.fileloading = "50";
      audio.onloadedmetadata = () => this.renderTime = audio.duration * (2.01-this.playback);
      this.play();
    },
    pause(){
      if (!this.fileurl) return this.error = 'upload audio first';
      this.offset = Tone.now() - this.startedAt;
      this.player.stop();
      this.isPlaying = false;
    },
    render(){
      if (!this.fileurl) return this.error = 'upload audio first';
      this.fileloading = "10";
      var renderedElm = document.createElement('a');
      Tone.Offline(async () => {
        this.fileloading = "40";
        const reverb = new Tone.Reverb({
                        decay: this.decay,
                        wet: this.wet,
                        preDelay: this.preDelay
                      }).toDestination();
        const song = new Tone.Player(this.fileurl).connect(reverb);
        song.playbackRate = this.playback;
        await Tone.loaded().then(()=>{ song.start(),this.fileloading = "70"; })
      }, this.renderTime).then((buffer) => {
        this.fileloading = "90";
        const wav = toWav(buffer);
        let blob  = new Blob([wav], {'type': "audio/wav"})
        let new_file = URL.createObjectURL(blob)
        renderedElm.href = new_file;
        renderedElm.download = this.renderFileName;
        renderedElm.style.display = "none";
        renderedElm.click();
        this.fileloading = false;
      })
    },
    play(){
      if (!this.fileurl) return this.error = 'upload audio first';
      if (this.isPlaying) return this.pause();
      const reverb = new Tone.Reverb({
        decay: this.decay,
        wet: this.wet,
        preDelay: this.preDelay
      }).toDestination();
      this.player = new Tone.Player(this.fileurl).connect(reverb);
      this.player.playbackRate = this.playback;
      this.player.loop = true;
      this.fileloading = "90";
      Tone.loaded().then(() => {
        this.fileloading = "100";
        this.startedAt = Tone.now() - this.offset
        this.player.start(0,this.offset);
        this.isPlaying = true;
        this.fileloading = false;
      });
    }
  }
}
</script>

<style>
.paper {
  background: rgba(255,255,255,0.6);
}
html {
  background: url(https://64.media.tumblr.com/2195bad5ac104715e0aeaec8664251ef/tumblr_p4h06pHPTH1wxdq3zo1_500.gif) no-repeat center center fixed;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;

}
.title{
  color: aliceblue;
  text-shadow: 2px 2px 3px #000000;
}
input[type=file] {
  overflow: hidden;
  text-overflow: clip;
}
</style>