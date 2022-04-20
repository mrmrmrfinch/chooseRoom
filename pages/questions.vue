<template>
  <v-app>
    <!-- Sizes your content based upon application components -->
    <v-main class="appMain">
      <div class='ripple-background' style="position:fixed;top:100%;  z-index:3;">
        <div class='circle xxlarge shade1'></div>
        <div class='circle xlarge shade2'></div>
        <div class='circle large shade3'></div>
        <div class='circle mediun shade4'></div>
        <div class='circle small shade5'></div>
      </div>
      <!-- Provides the application the proper gutter -->
      <v-container fluid>
        <!-- <a href="/"> Back to home </a> -->
        <v-container class="pageTitle">
        <h1 >Answer these questions to help us better understand your needs.</h1>
        </v-container>
        <!-- would automatically bind to questions -->
        <div class="questions">
        <div class="questionCard" v-for="(item, index) in questionsMap" :key="index">
          <p class="questionIndex">Question {{ index + 1 }} / {{questionsMap.length + 1}}</p>
          <p class="questionTitle"> {{ item.title }}</p>
          <p class="questionSubtitle">
            {{item.subtitle}}
          </p>
          <!-- If multiple choice -->
          <div v-if="item.type == 'multipleChoice'">
            <v-row style="justify-content: center;">
              <v-btn plain class="questionChoiceBtn" v-for="choice in item.choices" :key="choice.title" v-on:click="setQuestionChoice(index, choice.title, choice.weight)" :class="(item.selection === choice.title? 'questionBtnSelected' : 'questionBtnNormal')">
                <div>
                  <span class="material-icons questionIcon"> {{choice.icon}} </span
                  ><br />{{choice.title}}
                </div></v-btn
              >
            </v-row>
          </div>
          <!-- If slider -->
          <div v-if="item.type == 'slider'" style="width:100%">
            <v-container>
              <v-row>
                <v-col cols="2" style="text-align:right"><p class="questionSliderPrompt" >
                  <span class="material-icons questionSliderIcon">{{item.negativeChoice.icon}}</span>
                  {{item.negativeChoice.title}}</p></v-col>
                <v-col cols="8"> <v-slider max="100" min="-100" default="0" v-model="item.selectionWeight"></v-slider></v-col>
                <v-col cols="2">
                  <p class="questionSliderPrompt" style="text-align:left">{{item.positiveChoice.title}}
                    <span class="material-icons questionSliderIcon" style="width:24px">{{item.positiveChoice.icon}}</span>
                    </p></v-col>
              </v-row>
            </v-container>
            </div>
        </div>

        <div class="questionCard" >
          <p class="questionIndex">Question {{ questionsMap.length + 1 }} / {{questionsMap.length + 1}}</p>
          <p class="questionTitle">How many people at max would be in the room?</p>
          <p class="questionSubtitle">
            We need this information to filter out rooms that are too small.
          </p>
            <div style="width:100%">
            <v-container>
              <v-row>
                <v-col cols="2"> </v-col>
                <v-col cols="8"> <v-slider max="1000" min="1" default="100" v-model="minCapacity"></v-slider></v-col>
                <v-col cols="2">
                  <p class="questionSliderPrompt" style="text-align:left">
                    <span>{{ minCapacity }}</span>
                    </p></v-col>
              </v-row>
            </v-container>
            </div>
        </div>

          <v-container fluid style="margin-top:40px;">
            <v-container style="text-align:center;margin-bottom:30px;">
              <div style="display:flex; justify-content:center; width:100%">
              <h1 style="max-width:600px">
                How much does these questions matter in your decision making
                process?
              </h1>
              </div>
            </v-container>
            <div style="width:100%;">
          <div class="questionWeightSlider" v-for="(item, index) in questionsMap" :key=index>
            <div v-if="item.type == 'multipleChoice'">
              <v-row>
                <v-col cols="4" md="2">
              <p v-if="item.selection != ''">{{index + 1}}. {{item.shortTitle}}: {{item.selection}}</p>
              <p v-if="item.selection == ''">{{index + 1}}. {{item.shortTitle}}: Not Selected</p>
                </v-col>
                <v-col cols="8" md="10">
              <v-slider max="100" min="0" v-model="item.questionWeight"></v-slider>
                </v-col>
              </v-row>
            </div>
            <div v-if="item.type == 'slider'">
              <v-row>
                <v-col cols="4" md="2">
              <p>{{index + 1}}. {{item.shortTitle}}: {{item.selectionWeight / 2 + 50}}%</p>
                </v-col>
                <v-col cols="8" md="10">
              <v-slider max="100" min="0" v-model="item.questionWeight"></v-slider>
              </v-col>
              </v-row>
            </div>
          </div>
          </div>
          <v-btn style="width:100%;height:50px" v-on:click="submitPage">submit</v-btn>
          </v-container>
        </div>
        </div>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
const data = require("./questions.json");
export default {
  name: "QuestionPage",
  data() {
    return {
      questionsMap: data,
      minCapacity: 0,
    };
  },

  methods: {
    // clear every question's choice.
    clearAllSelections() {
      this.performanceType = "";
      this.roomCapacity = "";
      this.dryOrWet = "";
      this.requirePiano = "";
      this.requirePA = "";
    },

    // set the question's choice.
    setQuestionChoice(question, choice, weight) {
      this.questionsMap[question].selection = choice;
      this.questionsMap[question].selectionWeight = weight;
    },

    // submit the page.
    submitPage() {
      // check if all questions are answered.
      let allAnswered = true;
      let unanswered = [];
      for (let i = 0; i < this.questionsMap.length; i++) {
        if (this.questionsMap[i].selection == "") {
          // flag all unanswered questions' indices.
          unanswered.push(i + 1);
          allAnswered = false;
        }
      }
      // convert unanswered questions to string.
      let unansweredString = "";
      for (let i = 0; i < unanswered.length; i++) {
        if (i == 0) {
          unansweredString += "Question " + unanswered[i];
        } else {
          unansweredString += ", " + unanswered[i];
        }
      }
      // if not all answered, alert the user.
      if (!allAnswered) {
        alert(
          "You didn't answer: " +
            unansweredString +
            " Please answer all questions."
        );
        return;
      } else {
        // write to local storage, page has been submited.
        localStorage.setItem("submit", true);
        localStorage.setItem("minCapacity", this.minCapacity);
        this.$router.push("/reverb");
      }
    },
  },

  // when questionMap is updated, write to local storage.
  watch: {
    questionsMap: {
      handler: function (val, oldVal) {
        localStorage.setItem("questionsMap", JSON.stringify(val));
      },
      deep: true,
    },
  },
};
</script>

<style scoped>
@font-face {
  font-family: "Manrope";
  src: url("/manrope.woff2") format("woff2");
}

html,
body {
  margin: 0;
  padding: 0;
  font-family: "Manrope", sans-serif;
  background-color: #3399ff;
}

.appMain {
  margin: 0;
  padding: 0;
  font-family: "Manrope", sans-serif;
  background-color: #3399ff;
  height: 100vh;
  overflow: hidden;
}
@media only screen and (min-width: 600px) {
  .questions {
    position: absolute;
    right: 0;
    margin: 20px;
    padding: 40px;
    width: 70%;
    background-color: white;
    border-radius: 10px;
    padding: 10px;
    max-height: calc(100% - 40px);
    overflow: scroll;
  }

  .pageTitle {
    position: fixed;
    top: 20px;
    left: 30px;
    width: calc(30% - 70px);
    font-size: 1.2rem;
  }
}
@media only screen and (max-width: 601px) {
  .questions {
    width: calc(100% - 26px);
    background-color: white;
    border-radius: 10px;
    padding: 10px;
    padding-bottom: 50px;
    max-height: calc(100% - 100px);
    overflow: scroll;
    position: absolute;
    top: 130px;
    z-index: 999;
  }

  .pageTitle {
    position: fixed;
    top: 20px;
    left: 30px;
    height: 100px;
    width: calc(100% - 70px);
    font-size: 0.6rem;
    color: white;
    text-align: center;
  }
}

.questionCard {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding-top: 40px;
  padding-bottom: 40px;
  border-bottom: 1px dotted #ccc;
}

.questionTitle {
  font-size: 1.5rem;
  font-weight: bold;
  text-align: center;
  margin-bottom: 10px;
}

.questionSubtitle {
  font-size: 0.9rem;
  margin-bottom: 30px;
  text-align: center;
  opacity: 0.6;
}

.questionChoiceBtn {
  width: 200px;
  min-height: 160px;
  margin: 15px;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
}
.questionBtnNormal {
  color: black;
  background-color: #00bcd4;
  border-radius: 5px;
  outline: none;
}

.questionBtnSelected {
  color: white;
  background-color: green;
  border-radius: 10px;
}

.questionIcon {
  font-size: 80px;
  margin-bottom: 10px;
}

.questionSliderPrompt {
  font-size: 19px;
  display: flex;
  justify-content: center;
}

.questionSliderIcon {
  font-size: 20px;
}

.questionWeightSlider {
  width: 100%;
  padding-left: 20px;
  padding-right: 20px;
  margin-top: 10px;
  margin-bottom: 10px;
  text-align: right;
}

.circle {
  position: absolute;
  border-radius: 50%;
  background: white;
  animation: ripple 15s infinite;
  box-shadow: 0px 0px 1px 0px #508fb9;
}

.small {
  width: 200px;
  height: 200px;
  left: -100px;
  bottom: -100px;
}

.medium {
  width: 400px;
  height: 400px;
  left: -200px;
  bottom: -200px;
}

.large {
  width: 600px;
  height: 600px;
  left: -300px;
  bottom: -300px;
}

.xlarge {
  width: 800px;
  height: 800px;
  left: -400px;
  bottom: -400px;
}

.xxlarge {
  width: 1000px;
  height: 1000px;
  left: -500px;
  bottom: -500px;
}

.shade1 {
  opacity: 0.1;
}
.shade2 {
  opacity: 0.2;
}

.shade3 {
  opacity: 0.3;
}

.shade4 {
  opacity: 0.4;
}

.shade5 {
  opacity: 0.5;
}

@keyframes ripple {
  0% {
    transform: scale(0.8);
  }

  50% {
    transform: scale(1.2);
  }

  100% {
    transform: scale(0.8);
  }
}
</style>
