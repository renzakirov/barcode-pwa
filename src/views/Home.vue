<template lang="pug">
  div
    Button(type="success" @click="startScan" v-show="mode==0") SCAN
    Button(type="error" @click="stopScan" v-show="mode!==0") STOP
    h1(v-show="barcode") {{ barcode }}
    Scanner(v-if="mode===1" @stop-scan="stopScan")

</template>


<script>
import Scanner from '@/components/Scanner.vue';

export default {
  name: 'home',
  components: {
    Scanner,
  },
  data() {
    return {
      mode: 0,
      barcode: 0,
    }
  },
  methods: {
    startScan() {
      this.mode = 1;
    },
    stopScan(barcode = 0) {
      this.barcode = barcode;
      this.mode = 0;
      this.$Quagga.stop();
    },
  },
};
</script>
