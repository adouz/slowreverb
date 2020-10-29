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
      convolver: null,
      convolverGain: null,
      analyser: null,
      distortion: null,
      gainNode: null,
      biquadFilter: null,
      impulseNode: null,
      timeForAnImpulse: true,
      rate: 0.8
    }
  },
  mounted: function (){
    this.AudioContext =  new AudioContext();
    this.source = this.AudioContext.createBufferSource();
    this.convolverGain = this.AudioContext.createGain();
    this.convolver = this.AudioContext.createConvolver();
    this.analyser = this.AudioContext.createAnalyser();
    this.distortion = this.AudioContext.createWaveShaper();
    this.gainNode = this.AudioContext.createGain();
    this.biquadFilter = this.AudioContext.createBiquadFilter();
    // this.impulseNode = this.AudioContext.createScriptProcessor(2048,0,1);
    // this.impulseNode.onaudioprocess = this.impulseNodeCB;
    
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
    impulseNodeCB(evt){

        if(this.timeForAnImpulse){
            var buf = evt.outputBuffer.getChannelData(0);
            buf[0] = 1;
            this.timeForAnImpulse = false;
        } else {
            buf[0] = 0;
        }
    },
    loadImpulseResponse(){
      return ;
    },
    async play(){
      console.log(this.buffer);
      //const live = await this.getLiveAudio(this.AudioContext);
      if (this.AudioContext.state === 'suspended') this.AudioContext.resume();
      let buff = await this.AudioContext.decodeAudioData(this.buffer.slice(0));
      this.source.buffer = buff
      //this.convolver.buffer = await loadImpulseResponse();
      //this.convolverGain.gain.value = 0.5;
      
      this.source.playbackRate.value = this.rate; //pitch shifting
      //this.source.connect(this.convolverGain);
      this.source.connect(this.gainNode)
      this.gainNode.connect(this.AudioContext.destination)
      //this.source.connect(this.AudioContext.destination);
      
      this.source.start(0);
    }
  }
}
</script>

<style>
</style>