<template>
  <div id="app">
    <input type="file" @change="handlefile">
    <input type="submit" value="play" @click="play">
    <input type="submit" value="stop" @click="stop">
    <input type="text" name="rate" v-model="rate">
  </div>
</template>

<script>

export default {
  name: 'App',
  components: {
  },
  data (){
    return {
      audioContext: null,
      buffer: null,
      source: null,
      rate: 0.8
    }
  },
  mounted: function (){
    this.AudioContext =  new AudioContext();
    this.source = this.AudioContext.createBufferSource();
  },
  methods:{
    handlefile(e){
      const file = e.target.files[0];
      const reader = new FileReader();
      reader.onload = () => {
        this.buffer = reader.result; 
        this.play();
      }
      reader.readAsArrayBuffer(file)
    },
    stop(){
      this.source.disconnect();
      this.source.stop();
      this.AudioContext.suspend();
      this.source = this.AudioContext.createBufferSource();
    },
    async play(){
      console.log(this.buffer);
      if (this.AudioContext.state === 'suspended') this.AudioContext.resume();
      let buff = await this.AudioContext.decodeAudioData(this.buffer.slice(0));
      this.source.buffer = buff
      this.source.playbackRate.value = this.rate;
      this.source.connect(this.AudioContext.destination);
      this.source.start(0,this.AudioContext.currentTime);
    }
  }
}
</script>

<style>
</style>