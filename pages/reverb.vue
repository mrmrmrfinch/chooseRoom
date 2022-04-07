<template>
  <v-main>
    <v-app>
      <!-- Provides the application the proper gutter -->
      <v-container fluid>
        <p class="pageTitle">Reverb demo.</p>
        <a href="/"> Back to home </a>
        <div style="margin-top: 30px">
          <v-file-input
            label="Select Your Audio File"
            outlined
            dense
            @change="onAudioFileChange"
          ></v-file-input>
          <v-file-input
            label="Select Your IR File"
            outlined
            dense
            @change="onIRFileChange"
          ></v-file-input>
          <v-btn elevation="4" v-on:click="togglePlay"> Toggle Play</v-btn>
          <v-container class="px-0" fluid>
            <v-switch
              inset
              v-model="convolveSwitch"
              :label="`Convolve: ${convolveSwitch.toString()}`"
            ></v-switch>
          </v-container>
        </div>
      </v-container>
    </v-app>
  </v-main>
</template>

<script>
import * as Tone from "tone";

window.onclick = function () {
  Tone.start();
  Tone.context.lookAhead = 0;
};

export default {
  name: "reverbPage",
  data() {
    return {
      // initialize convolver and player variable
      player: null,
      audioURL: null,
      IRURL: null,
      convolveSwitch: true,
    };
  },
  // watch if convolveSwitch is changed, then change the convolver node statuss.
  watch: {
    convolveSwitch(val) {
      // if val, then convolver is on
      if (this.player) {
        if (val) {
          this.player.disconnect(0);
          this.player.connect(this.convolver);
        } else {
          this.player.disconnect(0);
          this.player.toMaster();
        }
      }
    },
  },

  methods: {
    togglePlay() {
      // if Web Audio is not initialized, initialize it
      if (!Tone.start) {
        Tone.start();
      }
      if (!Tone.context.state) {
        Tone.context.resume();
      }
      if (this.audioURL && this.IRURL) {
        this.player.connect(this.convolver);
        this.convolver.toMaster();

        // if audioPlayer is started, stop it
        if (this.player.state === "started") {
          this.player.stop();
          console.log("stopped");
        } else {
          this.player.start();
          console.log("start");
          console.log(this.player);
        }
      } else {
        console.log("no file selected");
      }
    },
    onAudioFileChange(fileURL) {
      // stop audio on change if the audio is playing.
      if (this.player !== null && this.player.state === "started") {
        this.player.stop();
      }
      if (fileURL) {
        this.audioURL = URL.createObjectURL(fileURL);
        this.player = new Tone.Player({
          url: this.audioURL,
          loop: true,
          volume: 6,
        });
      } else {
        this.player.stop();
      }
    },
    onIRFileChange(fileURL) {
      if (fileURL) {
        this.IRURL = URL.createObjectURL(fileURL);
        this.convolver = new Tone.Convolver({
          url: this.IRURL,
          wet: 1,
        });
      } else {
        this.convolver.disconnect();
      }
    },
  },

  mounted() {
    if (this.audioURL && this.IRURL) {
      this.player.connect(this.convolver);
      this.convolver.toMaster();
    } else {
      console.log("no file selected");
    }
  },
};
</script>
