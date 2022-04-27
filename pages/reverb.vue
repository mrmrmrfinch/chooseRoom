<template>
  <v-main>
    <v-app>
      <!-- Provides the application the proper gutter -->
      <v-container fluid>
        <v-btn depressed href="/"> back to home </v-btn>

        <v-container style="text-align: center">
          <h1>We think the best room for you is {{ bestRoom }}.</h1>
          <p>
            The next best two is {{ nextBestRoom }} ({{ nextBestProb }}% less
            likely) and {{ nextNextBestRoom }} ({{ nextNextBestRoomProb }}% less
            likely).
            <a href="#probChart"> Check our your probability for each room!</a>
          </p>
        </v-container>
        <v-container style="width: 80%; margin-left: 10%">
          <v-card
            elevation="10"
            style="padding: 30px; border-radius: 10px; min-height: 450px"
          >
            <h3>{{ bestRoom }}, and all other rooms' information are here.</h3>
            <v-row>
              <v-col cols="12" sm="8">
                <p class="explainText">
                  Although we would recommend {{ bestRoom }}, feel free to check
                  all rooms' data.
                </p>
              </v-col>
              <v-col cols="12" sm="4">
                <v-select
                  :items="IRArray"
                  @change="onRoomChange"
                  label="Select a room"
                  solo
                  style="width: 100%"
                ></v-select>
              </v-col>
            </v-row>
            <v-divider style="margin-bottom: 20px"></v-divider>
            <v-container
              fluid
              v-if="roomCapacity > 0"
              style="width: 100%; padding: 0; margin: 0"
            >
              <v-row>
                <v-col cols="12" sm="8">
                  <div style="margin-bottom: 25px">
                    <v-chip color="primary">
                      Capacity: {{ roomCapacity }}
                    </v-chip>
                    <v-chip color="primary"> PA: {{ roomhasPA }} </v-chip>
                    <v-chip color="primary"> Piano: {{ roomhasPiano }} </v-chip>
                  </div>
                  <p>{{ roomDescription }}</p>
                </v-col>
                <v-col cols="12" sm="4">
                  <v-card>
                    <img
                      :src="roomImage"
                      alt="room image"
                      width="100%"
                      style="padding: 10px"
                    />
                  </v-card>
                </v-col>
              </v-row>
            </v-container>
            <v-container
              fluid
              v-if="roomCapacity <= 0"
              style="text-align: center; color: grey; font-size: 10px"
            >
              <p style="margin-top: 100px">No Room Selected</p>
            </v-container>
          </v-card>
        </v-container>

        <v-container style="width: 80%; margin-left: 10%">
          <v-card elevation="10" style="padding: 30px; border-radius: 10px">
            <h3>You could hear how the rooms sound here.</h3>
            <v-row>
              <v-col cols="12" sm="8">
                <p class="explainText">
                  We are using Web Audio to convolve the room's impulse response
                  directly with the dry audio files that we provided to help you
                  get a sense of how the room will sound. To know more about how
                  we collected the impulse responses and what they mean, please
                  check out our <a href="">project writeup</a>.
                </p>
              </v-col>
              <v-col cols="12" sm="4">
                <v-container class="px-0" fluid>
                  <v-switch
                    inset
                    v-model="convolveSwitch"
                    :label="toDryWet(convolveSwitch)"
                  ></v-switch>
                </v-container>
              </v-col>
            </v-row>
            <v-row align="center">
              <v-col class="d-flex" cols="12" md="5" sm="6">
                <v-select
                  :items="drySoundsArray"
                  @change="onAudioFileChange"
                  label="Select a dry sound to try"
                  solo
                ></v-select>
              </v-col>

              <v-col class="d-flex" cols="12" md="5" sm="6">
                <v-select
                  :items="IRArray"
                  @change="onIRFileChange"
                  label="Select a room to try"
                  solo
                ></v-select>
              </v-col>

              <v-col class="d-flex" cols="12" md="2" sm="12">
                <v-btn
                  elevation="4"
                  v-on:click="togglePlay"
                  style="width: 100%; margin-top: -30px; height: 46px"
                >
                  Toggle Play</v-btn
                >
              </v-col>
            </v-row>
          </v-card>
        </v-container>
        <v-container style="width: 80%; margin-left: 10%">
          <v-card elevation="10" style="padding: 30px; border-radius: 10px">
            <h3>Here's your probability chart.</h3>
            <p class="explainText">
              The chart is calculated based on weights of each choice and
              properties of each room. We squash everything using tanh and
              sigmoid functions, then normalize the largest value to be one. If
              a room has a value of 1, it means it is the best room for you. If
              you notice some of the rooms have a very low probability, it's
              likely because they have a smaller capacity than the maximum
              number of people that will be present for your activity.
            </p>
            <div style="padding: 30px">
              <canvas
                class="d-none d-md-block"
                id="probChart"
                width="300"
                height="100"
                style="width: 80%"
              ></canvas>
              <canvas
                class="d-md-none"
                id="probChartMobile"
                width="100"
                height="150"
                style="width: 100%"
              ></canvas>
            </div>
          </v-card>
        </v-container>
      </v-container>
    </v-app>
  </v-main>
</template>

<script>
import * as Tone from "tone";
import Chart from "chart.js/auto";

// this is where we would store the audio files for IR and dry sounds
const roomIRLocation = "roomIR";
const drySoundsLocation = "drySounds";

const data = require("./rooms.json");
const dry = require("./drySounds.json");

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
      drySounds: dry,
      drySoundsArray: [],
      // bestRoom Calculated
      bestRoom: "",
      probabilityMap: {},
      sortedProbabilityMap: {},
      IRArray: [],
      // second bests
      nextBestRoom: "",
      nextBestProb: 0,
      nextNextBestRoom: "",
      nextNextBestRoomProb: 0,
      // init
      roomDescription: "",
      roomCapacity: 0,
      roomImage: "",
      roomhasPA: null,
      roomhasPiano: null,
      minCapacity: 0,
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
      Tone.start();
      Tone.context.lookAhead = 0;
      Tone.context.resume();
      if (this.audioURL && this.IRURL) {
        this.player.connect(this.convolver);
        this.convolver.toMaster();
        // if audioPlayer is started, stop it
        if (this.player.state === "started") {
          this.player.stop();
          console.log("stopped");
        } else {
          this.player.start();
        }
      } else {
        console.log("no file selected");
      }
    },

    onAudioFileChange(fileName) {
      let fileURL = "";

      // get fileURL from fileName in drySounds
      for (let i = 0; i < this.drySounds.length; i++) {
        if (this.drySounds[i].name === fileName) {
          fileURL = drySoundsLocation + "/" + this.drySounds[i].url;
        }
      }
      // stop audio on change if the audio is playing.
      if (this.player !== null && this.player.state === "started") {
        this.player.stop();
      }

      console.log(fileURL);

      if (fileURL !== "") {
        this.audioURL = fileURL;
        if (this.audioURL !== null) {
          this.player = new Tone.Player({
            url: this.audioURL,
            loop: true,
            volume: 6,
          });
          console.log(this.player);
        } else {
          this.player.stop();
          console.log("cannot get audioURL: " + this.audioURL);
        }
      } else {
        this.player.stop();
        console.log("no dry audio file selected");
      }
    },
    onIRFileChange(fileName) {
      Tone.start();
      Tone.context.lookAhead = 0;
      let fileURL = "";

      // get fileURL from fileName in roomsMap
      for (let i = 0; i < this.roomsMap.length; i++) {
        if (this.roomsMap[i].name === fileName) {
          fileURL = this.roomsMap[i].IRurl;
        }
      }
      if (fileURL) {
        this.IRURL = roomIRLocation + "/" + fileURL;
        console.log(this.IRURL);
        this.convolver = new Tone.Convolver({
          url: this.IRURL,
          wet: 1,
        });
      } else {
        if (this.convolver) {
          this.convolver.disconnect();
        }
        console.log(
          "cannot get IR file: " + fileName + " for room: " + fileName
        );
      }
    },
    onRoomChange(roomName) {
      // get roomName from roomsMap
      for (let i = 0; i < this.roomsMap.length; i++) {
        if (this.roomsMap[i].name === roomName) {
          this.roomDescription = this.roomsMap[i].description;
          this.roomCapacity = this.roomsMap[i].capacity;
          this.roomImage = this.roomsMap[i].image;
          if (this.roomsMap[i].hasPA > 0) {
            this.roomhasPA = "Yes";
          } else {
            this.roomhasPA = "No";
          }
          if (this.roomsMap[i].hasPiano > 0) {
            this.roomhasPiano = "Yes";
          } else {
            this.roomhasPiano = "No";
          }
        }
      }
    },
    toDryWet(bool) {
      if (bool) {
        return "Simulation";
      } else {
        return "Dry Sound";
      }
    },
  },

  mounted() {
    const vm = this;

    // check localStorage for submitted status.
    if (localStorage.getItem("submit")) {
      // get the questionsMap from local storage
      vm.minCapacity = localStorage.getItem("minCapacity");
      vm.questionsMap = JSON.parse(
        localStorage.getItem("questionsMap") || "{}"
      );
    } else {
      alert(
        "ERROR! You didn't submit from previous page. Please go to home page and start again!!!"
      );
    }

    for (let item in vm.drySounds) {
      vm.drySoundsArray.push(vm.drySounds[item].name);
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
          Math.tanh(this.roomsMap[room][linkedValue]) *
          (weight / 100) *
          (questionWeight / 100);
      }
    }

    // delete rooms in sorted Probability Map if they have capacity less than minCapacity
    for (let room in vm.roomsMap) {
      if (vm.roomsMap[room].capacity < vm.minCapacity) {
        // find the room in probability map
        for (let roomName in vm.probabilityMap) {
          if (vm.probabilityMap[roomName].name === vm.roomsMap[room].name) {
            vm.probabilityMap[roomName].prob = -100;
          }
        }
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

    // use sigmond function to squash the sorted probability map
    let squash = function (x) {
      return 1 / (1 + Math.exp(-x));
    };
    for (let room in vm.sortedProbabilityMap) {
      vm.sortedProbabilityMap[room].prob = squash(
        vm.sortedProbabilityMap[room].prob
      );
    }

    // normalize the sorted probability map so the largest value is 1.
    let max = 0;
    for (let room in vm.sortedProbabilityMap) {
      if (vm.sortedProbabilityMap[room].prob > max) {
        max = vm.sortedProbabilityMap[room].prob;
      }
    }
    for (let room in vm.sortedProbabilityMap) {
      vm.sortedProbabilityMap[room].prob /= max;
    }

    // get the next two best room and their probability
    vm.nextBestRoom = vm.sortedProbabilityMap[1].name;
    vm.nextBestProb = ((1 - vm.sortedProbabilityMap[1].prob) * 100).toFixed(2);
    vm.nextNextBestRoom = vm.sortedProbabilityMap[2].name;
    vm.nextNextBestRoomProb = (
      (1 - vm.sortedProbabilityMap[2].prob) *
      100
    ).toFixed(2);

    // convert sorted probability map to array with only name
    for (let item in vm.sortedProbabilityMap) {
      vm.IRArray.push(vm.sortedProbabilityMap[item].name);
    }

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
    const ctx2 = document.getElementById("probChartMobile");
    if (ctx2) {
      const probChart2 = new Chart(ctx2, {
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

<style>
.explainText {
  padding-top: 6px;
  font-size: 15px;
  opacity: 0.8;
}
</style>
