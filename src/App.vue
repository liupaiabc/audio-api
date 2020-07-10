<template>
    <div id="app">
        <span>melody</span><audio id="melody" controls src="./assets/melody.mp3"></audio>
        <span>harmonic</span><audio id="harmonic" controls src="./assets/bz.mp3"></audio>
        <span>maxgain</span><audio id="maxgain" controls src="./assets/ysq-new1.mp3"></audio>
        <button @click="useOscillatorNode">振荡器</button>
        <button @click="multinode">多轨一起播放</button>
        <button @click="useRawDataPlay">使用raw data</button>
        <button @click="useCompressor">使用压缩器</button>
        <button @click="useDelay">使用delay效果器</button>
        <button @click="useWaveShape">使用失真效果器</button>
        <button @click="useConvolver">使用混响效果器</button>
        <button @click="useStereoPanner">使用立体声效果器</button>
    </div>
</template>

<script>
export default {
  name: 'App',
  methods: {
    useOscillatorNode() {
        var audioCtx = new AudioContext();

        var oscillator = audioCtx.createOscillator();
        // 波形的类型
        oscillator.type = 'sine';
        // 声音的频率
        oscillator.frequency.value = 880;

        oscillator.connect(audioCtx.destination);

        oscillator.start();
    },
    multinode() {
        const audioCtx = new AudioContext();
        const audioElement = document.getElementById('melody');

        const audioElement1 = document.getElementById('harmonic');
        const sideTrack = audioCtx.createMediaElementSource(audioElement1);

        const gainNode = audioCtx.createGain();
        gainNode.gain.value = .8;
        sideTrack.connect(gainNode);
        gainNode.connect(audioCtx.destination);

        audioElement.play();
        audioElement1.play();
    },
    useRawDataPlay() {
      const audioCtx = new AudioContext();
      const request = new XMLHttpRequest();
      request.open('GET', 'https://static.yximgs.com/udata/pkg/frontend-explore/audio-test-sample.wav', true);
      request.responseType = 'arraybuffer';
      request.onload = function(e) {
          audioCtx.decodeAudioData(e.target.response, function(buffer) {
              const node = audioCtx.createBufferSource();
              node.loop = false;
              node.buffer = buffer;

              node.connect(audioCtx.destination);
              node.start();
          });
        };
      request.send();
    },
    useCompressor() {
        // 压缩器
        const audioCtx = new AudioContext();
        const audioElement = document.getElementById('maxgain');
        const track = audioCtx.createMediaElementSource(audioElement);

        var compressor = audioCtx.createDynamicsCompressor();
        // 音量超过多少压缩器开始工作
        compressor.threshold.setValueAtTime(-90, audioCtx.currentTime);
        // 压缩到多少
        compressor.ratio.setValueAtTime(20, audioCtx.currentTime);
        // 压缩器生效时间
        compressor.attack.setValueAtTime(0.003, audioCtx.currentTime);
        // 压缩器失效的时间
        compressor.release.setValueAtTime(0.25, audioCtx.currentTime);

        track.connect(compressor);
        compressor.connect(audioCtx.destination);

        audioElement.play();
    },
    useDelay() {
        // 延迟效果器
        const audioCtx = new AudioContext();
        const audioElement = document.getElementById('melody');
        const track = audioCtx.createMediaElementSource(audioElement);
        // 主声道
        const main = audioCtx.createGain();
        // 混响
        const echo = audioCtx.createGain();
        echo.gain.value = .7;
        // delay效果
        const delay = audioCtx.createDelay();
        delay.delayTime.value = .15;

        // 声音最后的输出
        const finalOutput = audioCtx.createGain();

        delay.connect(echo);
        echo.connect(delay);

        track.connect(delay);
        track.connect(main);

        main.connect(finalOutput);
        echo.connect(finalOutput);

        finalOutput.connect(audioCtx.destination);
        audioElement.play();
    },
    useWaveShape() {
        const audioCtx = new AudioContext();
        const audioElement = document.getElementById('harmonic');
        const track = audioCtx.createMediaElementSource(audioElement);
      
        const shaperNode = audioCtx.createWaveShaper();
        // gain 0-100
        const gain = 90;
        const n_samples = 44100;
        const curve = new Float32Array(n_samples);
        const deg = Math.PI / 180;
        let x = 0;
        for (var i = 0; i < n_samples; ++i ) {
            x = i * 2 / n_samples - 1;
            curve[i] = (3 + gain) * x * 20 * deg / (Math.PI + gain * Math.abs(x));
        }
        shaperNode.curve = curve;

        track.connect(shaperNode);
        shaperNode.connect(audioCtx.destination);

        audioElement.play();
    },
    useConvolver() {
        // 混响
        const audioCtx = new AudioContext();
        const audioElement = document.getElementById('melody');
        const track = audioCtx.createMediaElementSource(audioElement);
        // 主声道
        const main = audioCtx.createGain();
        // 这里可以设置各声道混合之后的效果
        main.gain.value = .3;
        // 混响
        const wet = audioCtx.createGain();
        wet.gain.value = 1.2;
        // 混响效果
        const convolver = audioCtx.createConvolver();
        const sampleRate = 44100;

        const length = sampleRate * 1.2;
        const impulse = audioCtx.createBuffer(2, length, sampleRate);
        const impulseL = impulse.getChannelData(0);
        const impulseR = impulse.getChannelData(1);

        const decay = 2;
        for (let i = 0; i < length; i++) {
            impulseL[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / length, decay);
            impulseR[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / length, decay);
        }
        convolver.buffer = impulse;
        // 声音最后的输出
        const finalOutput = audioCtx.createGain();

        convolver.connect(wet);

        track.connect(convolver);
        track.connect(main);

        main.connect(finalOutput);
        wet.connect(finalOutput);

        finalOutput.connect(audioCtx.destination);
        audioElement.play();
    },
    useStereoPanner() {
        // 使用立体声效果
        const audioCtx = new AudioContext();
        const audioElement = document.getElementById('melody');
        const track = audioCtx.createMediaElementSource(audioElement);

        const panNode = audioCtx.createStereoPanner();
        // value from [-1, 1]
        panNode.pan.setValueAtTime(1, audioCtx.currentTime);

        track.connect(panNode);
        panNode.connect(audioCtx.destination);

        audioElement.play();
    }
  }
}
</script>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}
button {
    display: block;
    font-size: 18px;
    margin: 0 auto 8px;
}
</style>
