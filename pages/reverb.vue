<template>
  <v-main>
    <v-app>
      <!-- Provides the application the proper gutter -->
      <v-container fluid>
        <a href="/"> Back to home </a>

        <v-container style="text-align: center">
          <h1>We think the best room for you is {{ bestRoom }}.</h1>
          <p>
            The next best two is {{ nextBestRoom }} ({{ nextBestProb }}% less
            likely) and {{ nextNextBestRoom }} ({{ nextNextBestRoomProb }}% less
            likely).
          </p>
        </v-container>
        <canvas id="probChart" width="400" height="100"></canvas>
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
import Chart from "chart.js/auto";
const data = require("./rooms.json");

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
      questionsMap: [],
      roomsMap: data,
      // bestRoom Calculated
      bestRoom: "",
      probabilityMap: {},
      sortedProbabilityMap: {},
      // second bests
      nextBestRoom: "",
      nextBestRoomProb: 0,
      nextNextBestRoom: "",
      nextNextBestRoomProb: 0,
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
    const vm = this;

    // check localStorage for submitted status.
    if (localStorage.getItem("submit")) {
      // get the questionsMap from local storage
      vm.questionsMap = JSON.parse(
        localStorage.getItem("questionsMap") || "{}"
      );
    } else {
      alert("ERROR! You didn't submit from previous page. Please go to home page and start again!!!")
    }

    if (this.audioURL && this.IRURL) {
      this.player.connect(this.convolver);
      this.convolver.toMaster();
    } else {
      console.log("no file selected");
    }

    // for each element in the questionsMap, multiply its weight to each room's corresponding linked property
    // put the result in a probability map
    for (let question in this.questionsMap) {
      let weight = this.questionsMap[question].selectionWeight;
      let questionWeight = this.questionsMap[question].questionWeight;
      let linkedValue = this.questionsMap[question].linkedValue;
      for (let room in this.roomsMap) {
        let roomName = this.roomsMap[room].name;
        if (vm.probabilityMap[roomName] === undefined) {
          vm.probabilityMap[roomName] = {
            prob: 1,
            name: roomName,
          };
        }
        vm.probabilityMap[roomName].prob +=
          this.roomsMap[room][linkedValue] *
          (weight / 100) *
          (questionWeight / 100);
      }
    }

    // find the best room
    let bestProb = 0;
    for (let room in vm.probabilityMap) {
      if (vm.probabilityMap[room].prob > bestProb) {
        bestProb = vm.probabilityMap[room].prob;
        vm.bestRoom = vm.probabilityMap[room].name;
      }
    }

    // normalize vm.probabilityMap, make min probability 0 and max probability 1.
    let min = 1;
    let max = 0;
    for (let room in vm.probabilityMap) {
      if (vm.probabilityMap[room].prob < min) {
        min = vm.probabilityMap[room].prob;
      }
      if (vm.probabilityMap[room].prob > max) {
        max = vm.probabilityMap[room].prob;
      }
    }
    for (let room in vm.probabilityMap) {
      vm.probabilityMap[room].prob =
        (vm.probabilityMap[room].prob - min) / (max - min);
    }

    // sort the probability map from high prob to low prob, with same data structure
    for (let room in vm.probabilityMap) {
      vm.sortedProbabilityMap[room] = {
        prob: vm.probabilityMap[room].prob,
        name: vm.probabilityMap[room].name,
      };
    }
    vm.sortedProbabilityMap = Object.values(vm.sortedProbabilityMap).sort(
      function (a, b) {
        return b.prob - a.prob;
      }
    );

    // get the next two best room and their probability
    vm.nextBestRoom = vm.sortedProbabilityMap[1].name;
    vm.nextBestProb = ((1 - vm.sortedProbabilityMap[1].prob) * 100).toFixed(2);
    vm.nextNextBestRoom = vm.sortedProbabilityMap[2].name;
    vm.nextNextBestRoomProb = (
      (1 - vm.sortedProbabilityMap[2].prob) *
      100
    ).toFixed(2);

    console.log(vm.sortedProbabilityMap);

    // charts
    const ctx = document.getElementById("probChart");
    if (ctx) {
      const probChart = new Chart(ctx, {
        type: "bar",
        data: {
          labels: Object.values(vm.sortedProbabilityMap).map(function (room) {
            return room.name;
          }),
          datasets: [
            {
              label: "Probability",
              data: Object.values(vm.sortedProbabilityMap).map(function (room) {
                return room.prob;
              }),
              backgroundColor: "rgba(255, 99, 132, 0.2)",
              borderColor: "rgba(255, 99, 132, 1)",
              borderWidth: 1,
            },
          ],
        },
        options: {
          scales: {
            y: {
              beginAtZero: true,
            },
          },
        },
      });
    }
  },
};
</script>
