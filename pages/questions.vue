<template>
  <v-app>
    <!-- Sizes your content based upon application components -->
    <v-main>
      <!-- Provides the application the proper gutter -->
      <v-container fluid>
        <a href="/"> Back to home </a>
        <v-container style="text-align:center">
        <h1 >Answer these questions to help us better understand your needs.</h1>
        </v-container>
        <!-- would automatically bind to questions -->

        <div class="questionCard" v-for="(item, index) in questionsMap" :key="index">
          <p class="questionIndex">Question {{ index + 1 }} / {{questionsMap.length}}</p>
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

          <v-container fluid style="margin-top:40px;">
            <v-container style="text-align:center;margin-bottom:30px;">
          <h1>
            How much does these questions matter in your decision making
            process?
          </h1>
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
      // Multiple Choice Questions
      performanceType: "",
      roomCapacity: "",
      dryOrWet: "",
      requirePiano: "",
      requirePA: "",
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
        alert("You didn't answer: " + unansweredString + " Please answer all questions.");
        return;
      } else {
      // write to local storage, page has been submited.
      localStorage.setItem("submit", true);
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
  color:white;
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

.quetsionSliderIcon {
  font-size: 20px;
}

.questionWeightSlider {
  width: 100%;
  padding-left:20px;
  padding-right:20px;
  margin-top: 10px;
  margin-bottom: 10px;
  text-align: right;
}
</style>
