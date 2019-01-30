<template lang="pug">

  div
    span(v-show="selectedCamera") {{ selectedCamera }}
    Button(type="primary" shape="circle" icon="ios-search" v-if="cameras && cameras.length>1" @click="nextCamera")

    h2(v-show="barcode") {{ barcode }}
    div#interactive.viewport
  
</template>

<script>
export default {
  name: 'Scanner',
  data() {
    return {
      barcode: '',
      cameras: [],
      selectedCamera: '',
      config: {
        inputStream: {
          name: 'Live',
          type: 'LiveStream',
        },
        decoder: {
          readers: ['code_128_reader'],
        },
        debug: {
          drawBoundingBox: true,
          showFrequency: false,
          drawScanline: true,
          showPattern: false,
        },
        multiple: false,
      },
    };
  },
  mounted() {
    this.$Quagga.onDetected(result => {
      var code = result.codeResult.code;
      console.log(code);
      this.barcode = code;
    });
    this.$Quagga.init(this.config, err => this.initCallback(err));
  },
  methods: {
    restart() {
      this.$Quagga.stop();
      this.$Quagga.init(this.config, err => this.initCallback(err));
    },
    initCallback(err) {
      if (err) {
        console.log(err);
        return;
      }
      this.initCameraSelection();
      console.warn('Initialization finished. Ready to start');
      this.$Quagga.start();
    },

    initCameraSelection() {
      this.selectedCamera = this.$Quagga.CameraAccess.getActiveStreamLabel();

      return this.$Quagga.CameraAccess.enumerateVideoDevices().then(devices => {
        function pruneText(text) {
          return text.length > 30 ? text.substr(0, 30) : text;
        }
        this.cameras = [];

        devices.forEach(device => {
          let cam = {};
          cam.value = device.deviceId || device.id;
          cam.label = pruneText(device.label || device.deviceId || device.id);
          cam.selected = this.selectedCamera === device.label;
          this.cameras.push(cam);
        });
        console.log('Cameras = ', this.cameras);
      });
    },

    nextCamera() {
      if (this.cameras && this.cameras.length > 1 && this.selectedCamera) {
        let index = this.cameras.findIndex(
          cam => cam.label === this.selectedCamera
        );
        if (index > -1) {
          index = index < this.cameras.length - 1 ? index + 1 : 0;
          this.config.constraints.deviceId = this.cameras[index].value;
          this.restart();
        }
      }
    },
  },
};
</script>
