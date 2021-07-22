<template>
  <div class="page">
    <div class="for-video">
      <video class="video" ref="video" :srcObject="stream" autoplay="autoplay">
      </video>
    </div>
    <div class="selectors">
      <div class="caption">Выбрать микрофон</div>
      <select class="select" v-model="audioinput">
        <option
          v-for="item in sortedDevices.audioinput"
          :key="item.deviceID"
          :disabled="item.disabled"
          :value="item"
        >
          {{ item.label }}
        </option>
      </select>

      <div class="caption">Выбрать видеокамеру</div>

      <select class="select" v-model="videoinput">
        <option
          v-for="item in sortedDevices.videoinput"
          :key="item.deviceID"
          :disabled="item.disabled"
          :value="item"
        >
          {{ item.label }}
        </option>
      </select>

      <div class="caption">Выбрать устройство вывода звука</div>
      
      <select class="select" v-model="audiooutput">
        <option
          v-for="item in sortedDevices.audiooutput"
          :key="item.deviceID"
          :disabled="item.disabled"
          :value="item"
        >
          {{ item.label }}
        </option>
      </select>
    </div>
  </div>
</template>

<script>
import webrtcAdapter from 'webrtc-adapter';
import '@/../node_modules/typeface-roboto/index.css';

export default {
  name: 'Page',
  components: {

  },
  data() {
    return {
      devices: null,
      sortedDevices: {
        'audioinput': [],
        'videoinput': [],
        'audiooutput': [],
      },
      audioinput: null,
      videoinput: null,
      audiooutput: null,
      stream: null,
      pause: false,
    }
  },
  mounted() {
    window.webrtcAdapter = webrtcAdapter;
    const constraints = {
      'video': true,
      'audio': true
    }
    navigator.mediaDevices.getUserMedia(constraints)
      .then(stream => {
      stream
        //console.log('Got MediaStream:', stream);
        navigator.mediaDevices.enumerateDevices().then(this.gotDevices).catch(this.handleError);
      })
      .catch(error => {
        console.error('Error accessing media devices.', error);
      });
  },
  watch: {
    devices(newDevs) {
      this.pause = true;
      this.sortedDevices.audioinput = newDevs.filter(dev => {
        return (dev.kind === 'audioinput')
      })
      this.sortedDevices.videoinput = newDevs.filter(dev => {
        return (dev.kind === 'videoinput')
      })
      this.sortedDevices.audiooutput = newDevs.filter(dev => {
        return (dev.kind === 'audiooutput')
      })
      // выбрать первые из списка
      this.audioinput = this.sortedDevices.audioinput ? this.sortedDevices.audioinput[0] : null
      this.videoinput = this.sortedDevices.videoinput ? this.sortedDevices.videoinput[0] : null
      this.audiooutput = this.sortedDevices.audiooutput ? this.sortedDevices.audiooutput[0] : null
      // задизаблить первые (дефолтные) значения
      this.sortedDevices.audioinput && this.sortedDevices.audioinput[0] && (this.sortedDevices.audioinput[0].disabled = true)
      this.sortedDevices.videoinput && this.sortedDevices.videoinput[0] && (this.sortedDevices.videoinput[0])
      this.sortedDevices.audiooutput && this.sortedDevices.audiooutput[0] && (this.sortedDevices.audiooutput[0].disabled = true)
      this.start();
    },
    audioinput() {
      if (this.pause) return;
      this.start();
    },
    videoinput() {
      if (this.pause) return;
      this.start();
    },
    audiooutput(newDev) {
      if (newDev) {
        this.changeAudioDestination(newDev && newDev.deviceId);
      }
    },
  },
  methods: {
    gotDevices(deviceInfos) {
      this.devices = deviceInfos;
      window.devices = deviceInfos;
    },

    // Attach audio output device to video element using device/sink ID.
    attachSinkId(element, sinkId) {
      if (!element) return
      if (typeof element.sinkId !== 'undefined') {
        element.setSinkId(sinkId)
            .then(() => {
              console.log(`Success, audio output device attached: ${sinkId}`);
            })
            .catch(error => {
              let errorMessage = error;
              if (error.name === 'SecurityError') {
                errorMessage = `You need to use HTTPS for selecting audio output device: ${error}`;
              }
              console.error(errorMessage);
              // Jump back to first output device in the list as it's the default.
              //audioOutputSelect.selectedIndex = 0;
            });
      } else {
        console.warn('Browser does not support output device selection.');
      }
    },

    changeAudioDestination(audioDestination) {
      if (!audioDestination) return;
      this.attachSinkId(this.$refs.video, audioDestination);
    },

    gotStream(stream) {
      window.stream = stream; // make stream available to console
      this.stream = stream;
      this.pause = false;
      //this.$refs.video.srcObject = stream;
      // Refresh button list in case labels have become available
      // return navigator.mediaDevices.enumerateDevices();
    },

    handleError(error) {
      console.log('navigator.MediaDevices.getUserMedia error: ', error.message, error.name);
    },

    start() {
      if (window.stream) {
        window.stream.getTracks().forEach(track => {
          track.stop();
        });
      }
      const audioSource = this.audioinput && this.audioinput.deviceId;
      const videoSource = this.videoinput && this.videoinput.deviceId;
       const constraints = {
         audio: {deviceId: audioSource ? {exact: audioSource} : undefined},
         video: {deviceId: videoSource ? {exact: videoSource} : undefined}
       };
      navigator.mediaDevices.getUserMedia(constraints).then(this.gotStream).catch(this.handleError);
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.page {
  display: flex;
  flex-direction: row;
}
.select {
  margin: 4px;
  padding: 10px;
  margin-left: 0px;
}
.for-video {
  width: 260px;
  margin-right: 20px;
  min-width: 260px;
}
.video {
  width: 100%;
  height: 100%;
}
.caption {
  padding: 0;
  padding-top: 4px;
  padding-bottom: 2px;
  font-family: "Roboto";
  font-size: 14px;
}
.selectors {
  display: flex;
  flex-direction: column;
}
</style>
