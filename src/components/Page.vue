<template>
  <div class="page">
    <div class="for-video">
      <div id="wavesurfer" ref="wavesurfer">
      </div>
      <video class="video" ref="video" :srcObject="stream" autoplay="autoplay">
      </video>
    </div>
    <div class="selectors">
      <div class="caption">Выбрать микрофон</div>
      <div class="for-selectors">
        <select class="select" :class="{ disabled: (audioinput ? audioinput.disabled : false), off: noaudioinput }" v-model="audioinput">
          <option
            v-for="item in sortedDevices.audioinput"
            :key="item.deviceID"
            :disabled="item.disabled"
            :value="item"
          >
            {{ item.label }}
          </option>
        </select>
        
        <label>
          <input class="checkbox" type="checkbox" v-model="noaudioinput" />
          <div class="chb"></div>
        </label>
      </div>

      <div class="caption">Выбрать видеокамеру</div>
      <div class="for-selectors">  
        <select class="select" :class="{ disabled: (audioinput ? videoinput.disabled : false), off: novideoinput}" v-model="videoinput">
          <option
            v-for="item in sortedDevices.videoinput"
            :key="item.deviceID"
            :disabled="item.disabled"
            :value="item"
          >
            {{ item.label }}
          </option>
        </select>
        <label>
          <input class="checkbox" type="checkbox" v-model="novideoinput" />
          <div class="chb"></div>
        </label>
      </div>

      <div class="caption">Выбрать устройство вывода звука</div>
      <div class="for-selectors">
        <select class="select" :class="{ disabled: (audioinput ? audiooutput.disabled : false), off: noaudiooutput }" v-model="audiooutput">
          <option
            v-for="item in sortedDevices.audiooutput"
            :key="item.deviceID"
            :disabled="item.disabled"
            :value="item"
          >
            {{ item.label }}
          </option>
        </select>
        <label>
          <input class="checkbox" type="checkbox" v-model="noaudiooutput" />
          <div class="chb"></div>
        </label>
      </div>

      <div class="caption">Выбрать пользователя</div>
      <div class="for-selectors">
        <select class="select" :class="{ disabled: (user ? user.disabled : false) }" v-model="user">
          <option
            v-for="(user, n) in users"
            :key="n"
            :disabled="user.disabled"
            :value="user"
          >
            {{ user.name }} 
            —
            {{ user.type }}
          </option>
        </select>
      </div>
    </div>
  </div>
  <div class="for-buttons"> 
    <button class="button" @click="addUser">
      Добавить в список
    </button>
    <div 
      class="user-item"
      v-for="(user, n) in users"
      :key="n"
      v-show="user.disabled"
      :ref="`user${user.id}`"
      @mouseenter = showPopper(user.id)
      @mouseleave = hidePopper(user.id)
    >
      {{ n+1 }}
      <div 
        class="tooltip"
        role="tooltip"
        :ref="`popper${user.id}`"
      >
        <div class="tooltip-item">
          Пользователь: {{ user.name }}
        </div>
        <div class="tooltip-item">
          Микрофон: {{ user.audioinput ? user.audioinput.label : 'нет' }}
        </div>
        <div class="tooltip-item">
          Камера: {{ user.videoinput ? user.videoinput.label : 'нет' }}
        </div>
        <div class="tooltip-item">
          Динамики: {{ user.audiooutput ? user.audiooutput.label : 'нет' }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import webrtcAdapter from 'webrtc-adapter';
import '@/../node_modules/typeface-roboto/index.css';
/* eslint-disable-next-line */
import WaveSurfer from'wavesurfer.js';
import Microphone from'wavesurfer.js/dist/plugin/wavesurfer.microphone';
import Popper from 'popper.js'
Popper.Defaults.modifiers.computeStyle.gpuAcceleration = false

import { v4 as uuidv4 } from 'uuid';

export default {
  name: 'Page',
  components: {
  },
  props: {
    users: {
      type: Object
    }
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
      playerOptions: {
        plugins: [
          Microphone.create(),
        ],
        height: 60,
      },
      wavesurfer: null,
      initialized: false,
      mic: null,
      user: null,
      noaudioinput: false,
      novideoinput: false,
      noaudiooutput: false,
      poppers: {},
    }
  },
  mounted() {
    // window.webrtcAdapter = webrtcAdapter;
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
    this.wavesurfer = WaveSurfer.create({
      container: '#wavesurfer', 
      ... this.playerOptions
    })

    let mic = this.wavesurfer.microphone

    mic.on('deviceReady', function(stream) {
      stream
      // console.log('Device ready!', stream);        
    });
    mic.on('deviceError', function(code) {
      console.warn('Device error: ' + code);
    });

    mic.start();

    this.mic = mic;

    this.users.forEach(user => {
      if (!user.id) {
        user.id = uuidv4()
      }
    })

    this.$emit('update:users', this.users)
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

      this.users && this.users[0] && (this.user = this.users[0])
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
  emits: ['update:users'],
  methods: {
    gotDevices(deviceInfos) {
      this.devices = deviceInfos;
      // window.devices = deviceInfos;
    },

    addUser() {

      if (this.videoinput.disabled && !this.novideoinput) {
        alert('Видеокамера уже занята, выберите другую');
        return;
      }

      if (this.audiooutput.disabled && !this.noaudiooutput) {
        alert('Устройство вывода звука уже занято, выберите другое')
        return;
      } 
      
      if (this.audioinput.disabled && !this.noaudioinput) {
        alert('Микрофон уже занят, выберите другой')        
        return;
      }

      if (this.user.disabled) {
        alert('Пользователь уже выбран, выберите другого')        
        return;
      }

      if (!this.novideoinput) {
        this.user.videoinput = this.videoinput
        this.videoinput.disabled = true;
      } else {
        this.user.videoinput = null
      }
       
      if (!this.noaudiooutput) {
        this.user.audiooutput = this.audiooutput
        this.audiooutput.disabled = true;
      } else {
        this.user.audiooutput = null
      }

      if (!this.noaudioinput) {
        this.user.audioinput = this.audioinput
        this.audioinput.disabled = true;
      } else {
        this.user.audioinput = null
      }

      this.user.disabled = true;
      this.$nextTick(function() {
        this.poppers[`user${this.user.id}`] =
        new Popper(
          this.$refs[`user${this.user.id}`],
          this.$refs[`popper${this.user.id}`],
          {
            placement: 'top',
            offset: [0, 20],
            strategy: 'fixed',
          }
        )

      });

      this.$emit('update:users', this.users)
    },

    showPopper(id) {
      this.$refs[`popper${id}`].setAttribute('data-show', '')
    },

    hidePopper(id) {
      this.$refs[`popper${id}`].removeAttribute('data-show')
    },

    // Attach audio output device to video element using device/sink ID.
    attachSinkId(element, sinkId) {
      if (!element) return
      if (typeof element.sinkId !== 'undefined') {
        element.setSinkId(sinkId)
            .then(() => {
              // console.log(`Success, audio output device attached: ${sinkId}`);
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
      // window.stream = stream; // make stream available to console
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
      if (this.stream) {
        this.stream.getTracks().forEach(track => {
          track.stop();
        });
      }
      const audioSource = this.audioinput && this.audioinput.deviceId;
      const videoSource = this.videoinput && this.videoinput.deviceId;
      const constraints = {
        audio: {deviceId: audioSource ? {exact: audioSource} : undefined},
        video: {deviceId: videoSource ? {exact: videoSource} : undefined}
      };

      this.mic.constraints = {
        ...constraints,
        video: false,
      };

      this.mic.start()
      navigator.mediaDevices.getUserMedia(constraints).then(this.gotStream).catch(this.handleError);
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.page {
  display: flex;
  flex-direction: row;
}
.select {
  margin: 4px;
  padding: 10px;
  margin-left: 0px;
  margin-right: 0px;
  flex-grow: 1;
  &.disabled {
    background-color: #debebe;
  }
  &.off {
    background-color: #ddd;
    color: #ccc
  }
}
.for-video {
  width: 260px;
  margin-right: 20px;
  min-width: 280px;
}
.video {
  width: 100%;
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
.button {
  background: none;
  padding: 10px;
  border: none;
  background-color: #669933;
}
.user-item {
  padding: 10px;
  margin-left: 10px;
  background-color: #669933;
  min-width: 16px;
  text-align: center;
}
.for-buttons {
  display: flex;
  flex-direction: row;
}
.checkbox {
  display: none;
  & ~ div.chb {
    height: 39px;
    width: 39px;
    background-color: #669933;
    margin-left: 10px;
  }
  &:checked ~ div.chb {
    background-color: #debebe;
  }
}
.for-selectors {
  display: flex;
  flex-direction: row;
  align-items: center;
}
.tooltip {
  background: #fff;
  color: #000;
  font-weight: normal;
  padding: 4px 8px;
  font-size: 14px;
  border: 2px dotted #ccc;
  border-radius: 4px;
  display: flex;
  flex-direction: column;
}
.tooltip-item {
  align-self: flex-start;
}
.tooltip {
  display: none;
}
.tooltip[data-show] {
  display: block;
}
</style>
