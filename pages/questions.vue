<template>
  <!-- Sizes your content based upon application components -->
  <v-main>
    <!-- Provides the application the proper gutter -->
    <v-container fluid>
      <p class="pageTitle">Question demo.</p>
      <a href="/"> Back to home </a>
      <div style="position: fixed; right: 10px; top: 10px">
        <v-card elevation="4" width="400">
          <v-card-title> Your Selections </v-card-title>
          <v-btn
            small
            elevation="0"
            v-on:click="clearAllSelections"
            style="position: absolute; top: 20px; right: 10px"
          >
            <v-icon>{{ icons.mdiClose }}</v-icon
            >Clear All</v-btn
          >
          <v-divider></v-divider>
          <v-card-text>
            Performance Type: {{ performanceType }} <br />
            Room Capacity: {{ roomCapacity }} <br />
            Dry or Wet: {{ dryOrWet }} <br />
            Require Piano: {{ requirePiano }} <br />
            Require PA: {{ requirePA }} <br />
          </v-card-text>
          <v-btn style="width:calc(100% - 20px);margin-left:10px;margin-bottom:20px;margin-top:20px"> Submit </v-btn>
        </v-card>
      </div>
      <div style="max-width: 500px">
        <v-radio-group row v-model="performanceType">
          <template v-slot:label>
            <div>What will you be performing?</div>
          </template>
          <v-radio value="speech">
            <template v-slot:label>
              <div>Speech/theater</div>
            </template>
          </v-radio>
          <v-radio value="music">
            <template v-slot:label>
              <div>Music (instrumental)</div>
            </template>
          </v-radio>
          <v-radio value="acappella">
            <template v-slot:label>
              <div>A Cappella</div>
            </template>
          </v-radio>
        </v-radio-group>

        <v-radio-group row v-model="roomCapacity">
          <template v-slot:label>
            <div>
              What is the required room capacity (performers and audience)?
            </div>
          </template>
          <v-radio value="1-20">
            <template v-slot:label>
              <div>1-20</div>
            </template>
          </v-radio>
          <v-radio value="25-50">
            <template v-slot:label>
              <div>25-50</div>
            </template>
          </v-radio>
          <v-radio value="50+">
            <template v-slot:label>
              <div>50+</div>
            </template>
          </v-radio>
        </v-radio-group>

        <v-radio-group row v-model="dryOrWet">
          <template v-slot:label>
            <div>Would you prefer a more dry or wet sounding room?</div>
          </template>
          <v-radio value="dry">
            <template v-slot:label>
              <div>Dry</div>
            </template>
          </v-radio>
          <v-radio value="wet">
            <template v-slot:label>
              <div>Wet</div>
            </template>
          </v-radio>
          <v-radio value="No preference">
            <template v-slot:label>
              <div>No preference</div>
            </template>
          </v-radio>
        </v-radio-group>

        <v-radio-group row v-model="requirePiano">
          <template v-slot:label>
            <div>Do you need a piano?</div>
          </template>
          <v-radio value="yes">
            <template v-slot:label>
              <div>Yes</div>
            </template>
          </v-radio>
          <v-radio value="no">
            <template v-slot:label>
              <div>No</div>
            </template>
          </v-radio>
        </v-radio-group>

        <v-radio-group row v-model="requirePA">
          <template v-slot:label>
            <div>Will you require a PA system?</div>
          </template>
          <v-radio value="yes">
            <template v-slot:label>
              <div>Yes</div>
            </template>
          </v-radio>
          <v-radio value="no">
            <template v-slot:label>
              <div>No</div>
            </template>
          </v-radio>
          <v-radio value="No preference">
            <template v-slot:label>
              <div>No preference</div>
            </template>
          </v-radio>
        </v-radio-group>
      </div>
    </v-container>
  </v-main>
</template>

<script>
import { mdiClose } from "@mdi/js";

export default {
  name: "QuestionPage",
  data() {
    return {
      // icons
      icons: {
        mdiClose,
      },
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
  },
};
</script>

<style scoped>
html,
body {
  margin: 0;
  padding: 0;
}
</style>
