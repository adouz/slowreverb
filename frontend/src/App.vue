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
    <input type="submit" value="render" @click="render">
    <input type="submit" value="play" ref="click">
      <a ref="rendered">rendered</a>
  </div>
</template>

<script>
import * as Tone from 'tone'
import * as toWav from 'audiobuffer-to-wav'
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
      renderFileName: 'none.wav'
    }
  },
  mounted: function (){
  },
  methods:{
    start(e){
      if (e.target.files){
        const file = e.target.files[0];
        this.fileurl =  URL.createObjectURL(file);
        this.renderFileName = "adouz "+file.name.split('.')[0];
      }
      if (this.player && this.player.state === "started") this.stop();
      this.offset = 0;
      this.startedAt = 0;
      const audio = new Audio();
      audio.src = this.fileurl;
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
      Tone.loaded().then(() => {
        this.startedAt = Tone.now() - this.offset
        this.player.start(0,this.offset);
      });
    }
  }
}
</script>

<style>
</style>