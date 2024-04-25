<template>
  <div>
  <div class="cameraPreview">
    <video id="cameraPreview" ref="cameraPreview" playsinline v-show="!imageClicked"></video>
    <img id="image" v-show="imageClicked" :src="imageSrc" ref="capturedImgRef" alt="Captured Image" />
  </div>
  <div class="clickButton" @click="takePicture">
  Take Picture</div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  mounted() {
    this.startCamera();
  },
  data() {
    return {
      imageSrc:"",
      imageClicked: false
    }
  },
  props: {
    msg: String
  },
  methods:{
    async startCamera(){
      const video = this.$refs.cameraPreview;
      this.mediaStream = await navigator.mediaDevices.getUserMedia({
        video:{
          width:{ideal:1024},
          facingMode:"environment",
          autoFocus:"true",
          aspectRatio:{ideal:4/5}
        }
      });
      video.srcObject = this.mediaStream;
      video.play();
      this.cameraPreview = true;
    },
    async closeCameraPreview(){
      return new Promise((resolve, reject) => {
          if(this.mediaStream){
            const tracks = this.mediaStream.getTracks();
            const promises = [];
            tracks.forEach(track => {
              promises.push(new Promise((trackResolve,trackReject) => {
                try{
                  track.stop();
                  this.mediaStream.removeTrack(track);
                  trackResolve();
                }catch(e){
                  trackReject(e);
                }
              }));
            });
      Promise.all(promises).then(() => {
        this.mediaStream = null;
        this.cameraPreview = false;
        resolve();
      }).catch(reject);
     }else{
      resolve();
      }
    });
    },
    async takePicture(){
      const video = this.$refs.cameraPreview;
      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d");
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
      const base64PictureData = canvas.toDataURL("image/png");
      await this.closeCameraPreview();
      this.imageSrc = base64PictureData;
      this.imageClicked = true;
    }
},
};
</script>
<style>
#cameraPreview{
  width: calc(100vw -30px);
}
</style>