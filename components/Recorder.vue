<template>
    <div class="container align-items-center">
      <div class="d-flex justify-content-center mt-5">
        <div class="card-custom col-sm-10 p-sm-4 p-3 m-sm-0 m-2 ">
            <div class="row">
                <div class="col-12">
                    <p>Click on start recording button to start recording and stop button to stop recording!</p>

                </div>            
            </div>
            <div class="row mt-3">
                <div class="col-8">
                <button class="btn btn-secondary" @click="startRecording" :disabled="recording">Start Recording</button>
                <button class="btn btn-secondary" @click="stopRecording" :disabled="!recording">Stop Recording</button>
            </div>
            <div class="col-4 ">
                <p class="float-end align-items-center" >Recording Time: {{ recordingTime }} secs</p>
            </div>
            </div>

            <div class="row mt-3">
                <div class="col-12">
                    <p>Your text will appear here shortly after you stop recording..</p>
                    <p>{{ recordedText }}</p>
                </div>
            </div>
            
         </div>
        </div>
      </div>
  </template>
  
  <script>
export default {
    data() {
    return {
        recordedText:"",
      mediaRecorder: null,
      chunks: [],
      recording: false,
      audioData: null,
      timerDuration: 10, // Timer duration in seconds
      recordingTime: 0, // Elapsed recording time
      timerInterval: null // Interval ID for the timer
    };
  },
  methods: {
    startRecording() {

        this.recordingTime = 0; // Reset the recording time
        this.timerInterval = setInterval(() => {
        this.recordingTime++;
      }, 1000); // Update the recording time every second

      navigator.mediaDevices.getUserMedia({ audio: true })
        .then(stream => {
          this.mediaRecorder = new MediaRecorder(stream);
          this.mediaRecorder.addEventListener('dataavailable', event => {
            this.chunks.push(event.data);
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

      clearInterval(this.timerInterval); // Stop the timer

      this.recording = false;
      this.mediaRecorder.addEventListener('stop', () => {
        const blob = new Blob(this.chunks, { type: 'audio/webm' });
        this.chunks = [];
        this.audioData = blob;
        
        let audioFile = new File([blob], 'recording.webm', { type: 'audio/webm' })

        console.log(audioFile);

        var myHeaders = new Headers();
        myHeaders.append("Authorization", "Bearer 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");

        var formdata = new FormData();
        formdata.append("file", audioFile, "recording.webm");
        formdata.append("model", "whisper-1");
        formdata.append("response_format", "json");

        var requestOptions = {
        method: 'POST',
        headers: myHeaders,
        body: formdata,
        redirect: 'follow'
        };

        fetch("https://api.openai.com/v1/audio/transcriptions", requestOptions)
        .then(response => response.text())
        .then(result => this.animateText(result))
        .catch(error => console.log('error', error));

         });
    },
    downloadRecording() {
      const url = URL.createObjectURL(this.audioData);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'recording.webm';
      a.click();
      URL.revokeObjectURL(url);
    },
    animateText(text1) {
      const speed = 100; // Typing speed in milliseconds
      let index = 0;

      const typingInterval = setInterval(() => {
        this.recordedText += text1[index];
        index++;

        if (index >= text1.length) {
          clearInterval(typingInterval);
        }
      }, speed);
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
  