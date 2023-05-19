<template>
    <div class="container align-items-center">
      <div class="d-flex justify-content-center">
        <div class="card-custom col-sm-10 p-sm-4 p-3 m-sm-0 m-2">
            <div>
                <button @click="startRecording" :disabled="recording">Start Recording</button>
                <button @click="stopRecording" :disabled="!recording">Stop Recording</button>
                <button @click="downloadRecording" :disabled="!recording || !audioData">Download</button>
                <audio ref="audioElement" controls></audio>
            </div>
         </div>
        </div>
      </div>
  </template>
  
  <script>
  export default {
  data() {
    return {
      mediaRecorder: null,
      audioChunks: [],
      recording: false,
      audioData: null
    };
  },
  methods: {
    startRecording() {
      navigator.mediaDevices.getUserMedia({ audio: true })
        .then(stream => {
          this.mediaRecorder = new MediaRecorder(stream);
          this.mediaRecorder.addEventListener('dataavailable', event => {
            this.audioChunks.push(event.data);
          });
          this.mediaRecorder.start();
          this.recording = true;
        })
        .catch(error => {
          console.error('Error accessing microphone:', error);
        });
    },
    stopRecording() {
      this.mediaRecorder.stop();
      this.recording = false;
      this.mediaRecorder.addEventListener('stop', () => {
        const audioBlob = new Blob(this.audioChunks);
        this.audioData = audioBlob;
        this.$refs.audioElement.src = URL.createObjectURL(audioBlob);
        console.log(audioBlob);
      });
    },
    downloadRecording() {
    if (this.audioData) {
        const url = URL.createObjectURL(this.audioData);
        const a = document.createElement("a");
        a.href = url;
        a.download = "recording.wav";
        a.click();
        URL.revokeObjectURL(url);
    }
  }
}
};
  </script>
  <style lang="css" scoped>
  .sub-title{
    color: gray;
    font-weight: 700;
  }
  p{
   font-weight: 500;
  }
  .card-custom{
  padding: 1rem  0.5rem;
  background-color: #ffffff;
  border-radius: 7px;
  box-shadow:  0 3px 10px rgb(154, 150, 150);
}


  </style>
  