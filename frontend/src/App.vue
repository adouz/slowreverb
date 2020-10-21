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
      detune
      <input type="number" min="0" max="100" step="10" v-model="detune">
    </p>
    <p>
      <input type="submit" value="start" @click="start">
    </p>
    <input type="file" @change="start">
    <input type="submit" value="play" @click="play">
    <input type="submit" value="stop" @click="stop">
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
      offset:0,
      startedAt:0,
      playback: 0.8,
      wet: 1,
      decay: 3,
      preDelay: 0.0,
      detune: 10
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
      console.log(Tone.now());
    },
    play(){
      const reverb = new Tone.Reverb({
        decay: this.decay,
        wet: this.wet,
        preDelay: this.preDelay
      }).toDestination();
      this.player = new Tone.GrainPlayer(this.fileurl).connect(reverb);
      this.player.playbackRate = this.playback;
      this.player.detune = this.detune;
      this.player.loop = true;
      Tone.loaded().then(() => {
        this.startedAt = Tone.now() - this.offset
        this.player.start(0,this.offset);
      });
      //player.autostart = true;
    }
  }
}
</script>

<style>
</style>