<template>
  <div id="app">
    <div class="text-center">
      <h1 class="title">slow + reverb</h1>
    </div>
    <div class="paper container container-sm margin-top-large">
      <div class="row flex-center margin-bottom-large">
        <input type="file" @change="start">
      </div>
      <div v-if="fileloading" class="progress margin-bottom">
        <div :class="'bar striped success w-'+fileloading"></div>
      </div>
      <div class="form-group">
        <label for="percentage">playback rate</label>
        <input class="input-block" type="range" step="0.01" min="0" max="1" v-model="playback" oninput="output.value = (this.value*100) + '%';">
        <output id="output" for="percentage">{{playback*100}}%</output>
      </div>
      <div class="form-group">
        <label for="percentage">reverb wet</label>
        <input class="input-block" type="range" step="0.01" min="0" max="1" v-model="wet" oninput="output.value = (this.value*100) + '%';">
        <output id="output" for="percentage">{{wet*100}}%</output>
      </div>
      <button class="btn-block" @click="start">START</button>
      <div class="row flex-center margin">
        <button class="btn-secondary-outline margin" @click="play">play</button>
        <button class="btn-success-outline margin" @click="stop">stop</button>
        <button class="btn-warning-outline margin" @click="render">render</button>
        <a ref="rendered" :class="`paper-btn btn-danger-outline margin ${isDownload ? '':'disabled'}`">Download</a>
      </div>
      <div class="row flex-center">
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
      isDownload: false
    }
  },
  mounted: function (){
  },
  methods:{
    start(e){
      if (e.target.files){
        this.fileloading = "20";
        const file = e.target.files[0];
        this.fileurl =  URL.createObjectURL(file);
        this.renderFileName = "adouz "+file.name.split('.')[0];
      }
      if (this.player && this.player.state === "started") this.stop();
      this.offset = 0;
      this.startedAt = 0;
      const audio = new Audio();
      audio.src = this.fileurl;
      this.fileloading = "50";
      audio.onloadedmetadata = () => {
        this.renderTime = audio.duration * (2.01-this.playback);
        console.log(audio.duration, this.renderTime)
      }
      this.play();
    },
    stop(){
      this.offset = Tone.now() - this.startedAt;
      this.player.stop();
    },
    render(){
      var renderedElm = this.$refs["rendered"], filename = this.renderFileName,
      playback = this.playback, decay = this.decay, wet = this.wet, 
      preDelay = this.preDelay, fileurl = this.fileurl;
      Tone.Offline(async function(){
        const reverb = new Tone.Reverb({
        decay: decay,
        wet: wet,
        preDelay: preDelay
      }).toDestination();
        const song = new Tone.Player(fileurl).connect(reverb);
        song.playbackRate = playback;
        await Tone.loaded().then(()=>{ song.start() })
      }, this.renderTime).then(function(buffer){
        const wav = toWav(buffer);
        let blob  = new Blob([wav], {'type': "audio/wav"})
        let new_file = URL.createObjectURL(blob)
        renderedElm.href = new_file;
        this.isDownload = true;
        renderedElm.download = filename;
      })
    },
    play(){
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
</style>