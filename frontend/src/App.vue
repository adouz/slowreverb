<template>
  <div id="app">
    <input type="file" @change="handlefile">
    <input type="submit" value="play" @click="play">
    <input type="submit" value="stop" @click="stop">
    <p>
      pitch rate
      <input type="number" min="0.1" max="1" step="0.1" v-model="rate">
      {{rate}}
    </p>
    <p>
      reverb wet
      <input type="number" min="0.1" max="1" step="0.1" v-model="wet">
      {{wet}}
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
      rate: 0.8,
      fileurl: null,
      player: null,
      wet: 1,
      offset:0,
      startedAt:0
    }
  },
  mounted: function (){
  },
  methods:{
    handlefile(e){
      const file = e.target.files[0];
      this.fileurl =  URL.createObjectURL(file);
      this.offset = 0;
      this.startedAt = 0;
      this.play();
    },
    stop(){
      this.offset = Tone.now() - this.startedAt;
      this.player.stop();
      console.log(Tone.now());
    },
    async play(){
      const reverb = new Tone.Reverb({
        decay: 2.5,
        wet: this.wet
      }).toDestination();
      this.player = new Tone.Player(this.fileurl).connect(reverb);
      this.player.playbackRate = this.rate;
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