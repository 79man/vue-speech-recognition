<script setup>
import { ref, onMounted } from "vue";

defineProps({
  msg: String,
});

const count = ref(0);
const transcript = ref("");
const isRecording = ref(false);
const debuglog = ref("");

const Recognition = window.SpeechRecognition || window.webkitSpeechRecognition;
const sr = new Recognition();

onMounted(() => {
  sr.contiuous = true;
  sr.interimResults = true;

  sr.onstart = () => {
    console.log("Voice Recog started");
    debuglog.value = "Voice Recog started";
    isRecording.value = true;
  };

  sr.onend = () => {
    console.log("Voice Recog stopped");
    debuglog.value = "Voice Recog stopped";
    isRecording.value = false;
  };

  sr.onresult = (evt) => {    
    transcript.value = Array.from(evt.results)
      .map((result) => result[0])
      .map((result) => annotateCommand(result.transcript))
      .join("");

    for (let i = 0; i < evt.results.length; i++) {
      const result = evt.results[i];

      if (result.isFinal) {
        debuglog.value = "Final Result Received";
        parseCommand(result);
      }
    }
  };
});

const annotateCommand = (transcript) => {
  let commands = [
    'stop recording',
    'what\'s the time',
    'what is the time',
    'command prompt'
  ];

  // Create a regular expression with all tokens separated by the | character
  const regex = new RegExp("\\b(" + commands.join("|") + ")\\b", "g");
  // Replace all occurrences of tokens with the marked tokens in the original string
  return transcript.replace(regex, '<b>$1</b>');  
}

const parseCommand = (result) => {

  let command = result[0].transcript;
  if(command.includes('stop recording')) {
    sr.stop();
  } else if(command.includes('what\'s the time') || command.includes('what is the time')) {
    sr.stop();
    transcript.value += ' The time is ' + (new Date()).toLocaleTimeString();
    setTimeout(() =>  sr.start(), 100);
  }
  else {
    sr.stop();
    setTimeout(() =>  sr.start(), 100);
  }
}

const toggleRecording = () => {
  if (isRecording.value) {    
    setTimeout(() => sr.stop(), 100);
  } else {    
    setTimeout(() => sr.start(), 100);
  }
};
</script>

<template>
  <div>
    <h1 class="message">{{ msg }}</h1>

    <div class="card">
      <button class="mic" @click="toggleRecording">
        {{ isRecording ? "Stop" : "Record" }}
      </button>
      <div class="transcript" v-html="transcript"></div>
      <div v-if="debuglog != ''" class="debuglog">{{ debuglog }}</div>
      <button type="button" @click="count++">count is {{ count }}</button>
    </div>
  </div>
</template>

<style scoped>
.debuglog {
  color: #888;
  padding: 10px;
  background-color: #eee;
  border-radius: 5px;
}
.message:hover {
  text-shadow: 5px -5px #78b49954;
}
</style>
