<template>
  <div id="app">
    <p>
      playback rate
      <input type="number" min="0" max="1" step="0.1" v-model="playback">
    </p>
    <p>
      reverb wet
      <input type="number" min="0" max="1" step="0.1" v-model="wet">
    </p>
    <p>
      reverb decay
      <input type="number" min="1" max="5" step="1" v-model="decay">
    </p>
    <p>
      reverb preDelay
      <input type="number" min="0" max="1" step="0.01" v-model="preDelay">
    </p>
    <p>
      <input type="submit" value="start" @click="start">
    </p>
    <input type="file" @change="start">
    <input type="submit" value="play" @click="play">
    <input type="submit" value="stop" @click="stop">
    <p>
      <audio ref="audio"></audio>
      <a ref="audioDownload">d</a>
    </p>
  </div>
</template>

<script>
import * as Tone from 'tone'
export default {
  name: 'App',
  components: {
  },
  data (){
    return {
      buffer: null,
      fileurl: null,
      player: null,
      madiaRec: null,
      offset:0,
      startedAt:0,
      playback: 0.8,
      wet: 0.6,
      decay: 2,
      preDelay: 0.0,
      chunks: []
    }
  },
  mounted: function (){
  },
  methods:{
    start(e){
      console.log(e)
      if (e.target.files){
        const file = e.target.files[0];
        this.fileurl =  URL.createObjectURL(file);
      }
      if (this.player && this.player.state === "started") this.stop();
      this.offset = 0;
      this.startedAt = 0;
      this.play();
    },
    stop(){
      this.offset = Tone.now() - this.startedAt;
      this.player.stop();
      this.madiaRec.stop();
      console.log(Tone.now());
    },
    play(){
      const reverb = new Tone.Reverb({
        decay: this.decay,
        wet: this.wet,
        preDelay: this.preDelay
      }).toDestination();
      this.player = new Tone.Player(this.fileurl).connect(reverb);
      reverb.connect(this.mediaStream());
      this.player.playbackRate = this.playback;
      this.player.loop = true;
      Tone.loaded().then(() => {
        this.startedAt = Tone.now() - this.offset
        this.player.start(0,this.offset);
        this.madiaRec.start();
      });
    },
    mediaStream(){
      const dest = Tone.context.createMediaStreamDestination();
      this.madiaRec = new MediaRecorder(dest.stream);
      this.madiaRec.ondataavailable = (e) => {this.chunks.push(e.data)}
      this.madiaRec.onstop = (e) => {
        e
        let audio = this.$refs["audio"];
        let Download = this.$refs["audioDownload"];
        audio.setAttribute('controls', '');
        audio.controls = true;
        let blob = new Blob(this.chunks, { 'type' : 'audio/mp3' });
        this.chunks = [];
        let audioURL = URL.createObjectURL(blob);
        audio.src = audioURL;
        Download.href=audioURL;
        Download.download= "audio.mp3";
      }
      return dest;
    }
  }
}
</script>

<style>
</style>