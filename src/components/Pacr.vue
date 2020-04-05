<template>
  <v-container fluid class="pa-50">
    <v-row class="text-center" align="center" justify="center">
      <v-col cols="12" md="4" sm="8">
        <v-card
          rounded
          class="white pa-4"
          style=".transparent {
                background-color: white!important;
                opacity: 0.9;
                border-color: transparent!important;
              }"
        >
          <!--<v-card-title>Calculate your pace, time or distance!</v-card-title>-->
          <v-form>
            <v-container class="pa-0 pt-2 pb-2">
              <v-row align="center" no-gutters>
                <v-col cols="3">
                  <v-badge :value="time.calculated" color="green" dot overlap>
                    <v-tooltip left fixed>
                      <template v-slot:activator="{ on }">
                        <v-btn
                          rounded
                          fab
                          x-small
                          elevation="1"
                          color="primary"
                          v-on="on"
                          @click="calculateTime"
                        >
                          <v-icon>mdi-equal</v-icon>
                        </v-btn>
                      </template>
                      <span>Calculate your time</span>
                    </v-tooltip>
                  </v-badge>
                </v-col>
                <v-col>
                  <v-text-field
                    type="text"
                    label="Time"
                    ref="timeTextField"
                    v-model="time.input"
                    outlined
                    dense
                    autocomplete="off"
                    :error-messages="time.errMessage"
                    @change="eventTimeChanged"
                    @input="eventTimeInput"
                    @focus="eventTimeFocus"
                    :placeholder="getTimePlaceholder"
                    @keydown="eventTimeKey"
                  />
                </v-col>
              </v-row>
            </v-container>
            <v-container class="pa-0 pt-2 pb-2">
              <v-row align="center" no-gutters>
                <v-col cols="3">
                  <v-badge :value="dist.calculated" color="green" dot overlap>
                    <v-tooltip left>
                      <template v-slot:activator="{ on }">
                        <v-btn
                          rounded
                          fab
                          x-small
                          elevation="1"
                          v-on="on"
                          color="primary"
                          @click="calculateDistance"
                        >
                          <v-icon>mdi-equal</v-icon>
                        </v-btn>
                      </template>
                      <span>Calculate your distance</span>
                    </v-tooltip>
                  </v-badge>
                </v-col>
                <v-col>
                  <v-menu offset-x>
                    <template v-slot:activator="{ on }">
                      <v-text-field
                        type="text"
                        outlined
                        dense
                        label="Distance"
                        ref="distanceTextField"
                        v-model="dist.input"
                        autocomplete="off"
                        :suffix="distanceUnit"
                        :error-messages="dist.errMessage"
                        append-icon="mdi-dots-vertical"
                        @change="eventDistanceChanged"
                        @click:append="on.click"
                        @input="eventDistanceInput"
                        @keydown="eventDistanceKey"
                      />
                    </template>
                    <v-list dense nav>
                      <v-subheader>UNITS</v-subheader>
                      <v-list-item-group mandatory value="Marathon">
                        <v-list-item
                          v-for="(item, index) in dist.units"
                          :key="index"
                          @click="clickPresetUnits(index)"
                        >
                          <v-list-item-title>{{ item }}</v-list-item-title>
                        </v-list-item>
                      </v-list-item-group>
                      <v-divider></v-divider>
                      <v-list-item-group v-model="distanceChoice">
                        <v-subheader>DISTANCES</v-subheader>
                        <v-list-item
                          v-for="(item, index) in presetDistances"
                          :key="index"
                          @click="clickPresetDistance(index)"
                        >
                          <v-list-item-title>{{ item.title }}</v-list-item-title>
                        </v-list-item>
                      </v-list-item-group>
                    </v-list>
                  </v-menu>
                </v-col>
              </v-row>
            </v-container>
            <v-container class="pa-0 pt-2 pb-2">
              <v-row align="center" no-gutters>
                <v-col cols="3" align="center">
                  <v-badge :value="pace.calculated" color="green" dot overlap>
                    <v-tooltip left>
                      <template v-slot:activator="{ on }">
                        <v-btn
                          rounded
                          fab
                          x-small
                          elevation="1"
                          color="primary"
                          @click="calculatePace"
                          v-on="on"
                        >
                          <v-icon>mdi-equal</v-icon>
                        </v-btn>
                      </template>
                      <span>Calculate your pace</span>
                    </v-tooltip>
                  </v-badge>
                </v-col>
                <v-col>
                  <v-spacer></v-spacer>
                  <v-text-field
                    type="text"
                    label="Pace"
                    outlined
                    dense
                    ref="paceTextField"
                    v-model="pace.input"
                    autocomplete="off"
                    persistent-hint
                    :suffix="paceUnit"
                    :error-messages="pace.errMessage"
                    append-icon="mdi-unfold-more-horizontal"
                    @change="eventPaceChanged"
                    @click:append="togglePaceUnits"
                    @input="eventPaceInput"
                    @keydown="eventPaceKey"
                  />
                </v-col>
              </v-row>
            </v-container>
          </v-form>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn small elevation="2" @click="reset">Reset</v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import * as units from "../const";

export default {
  name: "Pacr",
  data: () => ({
    presetDistances: [
      { title: "Marathon", value: 42.2, unit: units.km },
      { title: "Half marathon", value: 21.1, unit: units.km },
      { title: "10k", value: 10, unit: units.km },
      { title: "5k", value: 5, unit: units.km },
      { title: "800m", value: 800, unit: units.m },
      { title: "400m", value: 400, unit: units.m }
    ],
    time: {
      validated: false,
      value: 0,
      input: "",
      errMessage: "",
      focused: false,
      calculated: false
    },
    dist: {
      validated: false,
      value: 0,
      units: [units.km, units.mi, units.m],
      unitIndex: 0,
      input: "",
      errMessage: "",
      calculated: false,
      step: 0.1
    },
    pace: {
      validated: false,
      units: [units.min_km, units.min_mi],
      unitIndex: 0,
      value: 0,
      input: "",
      errMessage: "",
      calculated: false
    },
    distanceChoice: ""
  }),

  computed: {
    distanceUnit() {
      return this.dist.units[this.dist.unitIndex];
    },
    paceUnit() {
      return this.pace.units[this.pace.unitIndex];
    },
    timeOuterIcon() {
      // return time.validated ? 'mdi-check' : 'mdi-blank'
      return "mdi-blank";
    },
    distanceOuterIcon() {
      // return dist.validated ? 'mdi-check' : 'mdi-blank'
      return "mdi-dots-vertical";
    },
    paceOuterIcon() {
      // return pace.validated ? 'mdi-check' : 'mdi-blank'
      return "mdi-blank";
    },
    getTimePlaceholder() {
      if (this.time.focused) {
        return "00:00:00";
      }
      return "";
    },
    bgHeight() {
      return this.$vuetify.breakpoint.height - 95;
    }
  },

  methods: {
    toggleDistanceUnits() {
      if (this.dist.unitIndex + 1 >= this.dist.units.length) {
        this.dist.unitIndex = 0;
      } else {
        this.dist.unitIndex++;
      }
    },
    togglePaceUnits() {
      if (this.pace.unitIndex + 1 >= this.pace.units.length) {
        this.pace.unitIndex = 0;
      } else {
        this.pace.unitIndex++;
      }
    },
    clickPresetDistance(index) {
      // set the distance value
      this.dist.input = this.presetDistances[index].value;
      // set the units
      if (
        this.presetDistances[index].unit !==
        this.dist.units[this.dist.unitIndex]
      ) {
        // change the units...
        //console.log("search for units " + this.presetDistances[index].unit);
        for (let i = 0; i < this.dist.units.length; i++) {
          if (this.presetDistances[index].unit === this.dist.units[i]) {
            this.dist.unitIndex = i;
            //console.log("found at " + i);
            break;
          }
        }
      }

      // call the input event
      this.eventDistanceInput(this.dist.input);
    },
    clickPresetUnits(index) {
      // set the units
      this.dist.unitIndex = index;
    },
    clearErrorMessage(obj) {
      obj.errMessage = "";
    },

    eventTimeChanged(input) {
      //console.log("time changed! " + this.time.input);
      if (input === "") {
        // reset the userInput
        this.time.userInput = false;
        return;
      }

      // validate the entry
      this.time.userInput = true;
      /*
      if (this.dist.userInput && !this.pace.userInput) {
        this.calculatePace();
      } else if (!this.dist.userInput && this.pace.userInput) {
        this.calculateDistance();
      }*/
    },
    eventDistanceChanged(input) {
      //console.log("distance changed! " + this.dist.input);
      if (input === "") {
        // reset the userInput
        this.dist.userInput = false;
        return;
      }
      this.dist.userInput = true;
      /*
      // if one (and only one) of the other parameters are set: calculate the other
      if (this.time.userInput && !this.pace.userInput) {
        this.calculatePace();
      } else if (!this.time.userInput && this.pace.userInput) {
        this.calculateTime();
      }*/
    },
    eventPaceChanged(input) {
      //console.log("pace changed! " + this.pace.input);
      if (input === "") {
        // reset the userInput
        this.pace.userInput = false;
        return;
      }
      this.pace.userInput = true;
      /*
      // if one (and only one) of the other parameters are set: calculate the other
      if (this.time.userInput && !this.dist.userInput) {
        this.calculateDistance();
      } else if (!this.time.userInput && this.dist.userInput) {
        this.calculateTime();
      }*/
    },
    eventTimeInput(input) {
      //console.log("time input: " + input);
      this.time.calculated = false;

      // clear the message error if the value is valid
      if (this.time.errMessage !== "") {
        if (this.validateTimeStr(input)) {
          this.clearErrorMessage(this.time);
        }
      }
    },
    eventTimeFocus() {
      this.time.focused = true;
    },
    eventTimeKey(KeyboardEvent) {
      //console.log(KeyboardEvent);

      // Enter: try to calculate the third variable
      if (KeyboardEvent.key === "Enter") {
        this.triggerCalculation();
        return;
      }

      // allow arrow up or down to increment/decrement value
      let step;
      if (KeyboardEvent.key === "ArrowUp") {
        step = 1;
      } else if (KeyboardEvent.key === "ArrowDown") {
        step = -1;
      } else {
        return;
      }

      // check if the input value is valid before increment
      let input;
      if (this.time.input === "") {
        // empty input: assume 0.0
        input = 0.0;
      } else if (this.validateTimeStr(this.time.input)) {
        input = this.inputToSec(this.time.input);
      } else {
        //console.log("time " + this.time.input + " not valid, abort increment");
        return;
      }

      // minimum : 0.0
      if (input + step >= 0.0) {
        // increment the value by step
        this.time.input = this.secToTimeStrWithHour(input + step);
        this.eventTimeInput(this.time.input);
      }
    },
    eventDistanceInput(input) {
      //console.log("distance input: " + input);
      this.dist.calculated = false;

      // clear the message error if the value is valid
      if (this.dist.errMessage !== "") {
        if (this.validateFloatStr(input)) {
          this.clearErrorMessage(this.dist);
        }
      }
    },
    eventDistanceKey(KeyboardEvent) {
      //console.log(KeyboardEvent);

      // tab: next text field: not working
      /*if (KeyboardEvent.key === "Tab") {
        this.$refs.paceTextField.focus();
        return;
      }*/

      // Enter: try to calculate the third variable
      if (KeyboardEvent.key === "Enter") {
        this.triggerCalculation();
        return;
      }

      // allow arrow up or down to increment/decrement value
      let step;
      if (KeyboardEvent.key === "ArrowUp") {
        step = this.dist.step;
      } else if (KeyboardEvent.key === "ArrowDown") {
        step = -1 * this.dist.step;
      } else {
        return;
      }

      // check if the input value is valid before increment
      let input;
      if (this.dist.input === "") {
        // empty input: assume 0.0
        input = 0.0;
      } else if (this.validateFloatStr(this.dist.input)) {
        input = this.inputToFloat(this.dist.input);
      } else {
        /*console.log(
          "distance " + this.dist.input + " not valid, abort increment"
        );*/
        return;
      }

      //console.log("input: " + input + ", step: " + step);

      // minimum : 0.0
      if (input + step >= 0.0) {
        // increment the value by step
        this.dist.input = (input + step).toFixed(1);
        this.eventDistanceInput(this.dist.input);
      }
    },
    eventPaceInput(input) {
      //console.log("pace input: " + input);
      this.pace.calculated = false;

      // clear the message error if the value is valid
      if (this.pace.errMessage !== "") {
        if (this.validateTimeStr(input)) {
          this.clearErrorMessage(this.pace);
        }
      }
    },
    eventPaceKey(KeyboardEvent) {
      //console.log(KeyboardEvent);

      // Enter: try to calculate the third variable
      if (KeyboardEvent.key === "Enter") {
        this.triggerCalculation();
        return;
      }

      // not working
      /*
      if (KeyboardEvent.key === "Tab") {
        this.$refs.timeTextField.focus();
        return;
      }
      */

      // allow arrow up or down to increment/decrement value
      let step;
      if (KeyboardEvent.key === "ArrowUp") {
        step = 1;
      } else if (KeyboardEvent.key === "ArrowDown") {
        step = -1;
      } else {
        return;
      }

      // check if the input value is valid before increment
      let input;
      if (this.pace.input === "") {
        // empty input: assume 0.0
        input = 0.0;
      } else if (this.validateTimeStr(this.pace.input)) {
        input = this.inputToSec(this.pace.input);
      } else {
        //console.log("pace " + this.pace.input + " not valid, abort increment");
        return;
      }

      // minimum : 0.0
      if (input + step >= 0.0) {
        // increment the value by step
        this.pace.input = this.secToTimeStr(input + step);
        this.eventPaceInput(this.pace.input);
      }
    },

    calculateTime() {
      // time[sec] = pace[sec/km] * distance[km]

      // first, clear the error message on the current item
      if (this.time.errMessage !== "") {
        this.time.errMessage = "";
      }

      let validParameters = true;

      // validate pace input
      if (!this.validateTimeStr(this.pace.input)) {
        // set error message for pace
        this.pace.errMessage = "Enter a valid pace (mm:ss)";
        validParameters = false;
      }

      if (!this.validateFloatStr(this.dist.input)) {
        // set error message for distance
        this.dist.errMessage = "Enter a valid distance";
        validParameters = false;
      }

      if (!validParameters) {
        return;
      }

      let paceSec = this.inputToSec(this.paceMinKm(this.pace));
      let distKm = this.inputToFloat(this.distanceKm(this.dist));

      // validate: not zero values
      if (paceSec === 0) {
        this.pace.errMessage = "Enter a valid pace (mm:ss)";
        validParameters = false;
      }

      if (distKm === 0) {
        this.dist.errMessage = "Enter a valid distance";
        validParameters = false;
      }

      if (!validParameters) {
        return;
      }

      let timeSec = paceSec * distKm;

      this.time.input = this.secToTimeStrWithHour(timeSec);
      this.time.calculated = true;

      // reset others (already validated)
      if (this.dist.calculated) {
        this.dist.calculated = false;
        this.dist.userInput = true;
      }

      if (this.pace.calculated) {
        this.pace.calculated = false;
        this.pace.userInput = true;
      }
      /*
      console.log(
        "calculate time with pace: " +
          this.pace.input +
          " (" +
          paceSec +
          " sec) and distance: " +
          this.dist.input +
          " (" +
          distKm +
          " km) = " +
          this.time.input +
          " (" +
          timeSec +
          " sec)"
      );*/
    },
    calculateDistance() {
      // distance[km] = time[sec] / pace[sec/km]

      // first, clear the error message on the current item
      if (this.dist.errMessage !== "") {
        this.dist.errMessage = "";
      }

      let validParameters = true;
      // validate time input
      if (!this.validateTimeStr(this.time.input)) {
        // set error message for time
        this.time.errMessage = "Enter a valid time (hh:mm:ss or mm:ss)";
        validParameters = false;
      }

      // validate pace input
      if (!this.validateTimeStr(this.pace.input)) {
        // set error message for pace
        this.pace.errMessage = "Enter a valid pace (mm:ss)";
        validParameters = false;
      }

      if (!validParameters) {
        return;
      }

      let paceSec = this.inputToSec(this.paceMinKm(this.pace));
      let timeSec = this.inputToSec(this.time.input);

      // validate: not zero values
      if (paceSec === 0) {
        this.pace.errMessage = "Enter a valid pace (mm:ss)";
        validParameters = false;
      }

      if (timeSec === 0) {
        this.time.errMessage = "Enter a valid time (mm:ss)";
        validParameters = false;
      }

      if (!validParameters) {
        return;
      }

      let distKm = timeSec / paceSec;

      // convert to used units
      if (this.dist.units[this.dist.unitIndex] === units.m) {
        distKm *= 1000.0;
      } else if (this.dist.units[this.dist.unitIndex] === units.mi) {
        distKm /= 1.609344;
      }

      this.dist.input = distKm.toFixed(1);
      this.dist.calculated = true;

      // reset others (already validated)
      if (this.time.calculated) {
        this.time.calculated = false;
        this.time.userInput = true;
      }

      if (this.pace.calculated) {
        this.pace.calculated = false;
        this.pace.userInput = true;
      }
      /*
      console.log(
        "calculate distance with time: " +
          this.time.input +
          " (" +
          timeSec +
          " sec) and pace: " +
          this.pace.input +
          " (" +
          paceSec +
          " sec) = " +
          this.dist.input +
          " (" +
          distKm +
          " km)"
      );
      */
    },
    calculatePace() {
      // pace[sec/km] = time[sec] / distance[km]

      // first, clear the error message on the current item
      if (this.pace.errMessage !== "") {
        this.pace.errMessage = "";
      }

      let validParameters = true;

      // validate time input
      if (!this.validateTimeStr(this.time.input)) {
        // set error message for time
        this.time.errMessage = "Enter a valid time (hh:mm:ss or mm:ss)";
        validParameters = false;
      }

      if (!this.validateFloatStr(this.dist.input)) {
        // set error message for distance
        this.dist.errMessage = "Enter a valid distance";
        validParameters = false;
      }

      if (!validParameters) {
        return;
      }

      let timeSec = this.inputToSec(this.time.input);
      let distKm = this.inputToFloat(this.distanceKm(this.dist));

      // validate: not zero values
      if (distKm === 0) {
        this.dist.errMessage = "Enter a valid distance";
        validParameters = false;
      }

      if (timeSec === 0) {
        this.time.errMessage = "Enter a valid time (mm:ss)";
        validParameters = false;
      }

      if (!validParameters) {
        return;
      }

      let paceSec = Math.floor(timeSec / distKm);

      if (this.pace.units[this.pace.unitIndex] === units.min_mi) {
        paceSec *= 1.609344;
      }

      this.pace.input = this.secToTimeStr(paceSec);
      this.pace.calculated = true;

      // reset others (already validated)
      if (this.dist.calculated) {
        this.dist.calculated = false;
        this.dist.userInput = true;
      }

      if (this.time.calculated) {
        this.time.calculated = false;
        this.time.userInput = true;
      }
      /*
      console.log(
        "calculate pace with time: " +
          this.time.input +
          " (" +
          timeSec +
          " sec) and distance: " +
          this.dist.input +
          " (" +
          distKm +
          " km) = " +
          this.pace.input +
          " (" +
          paceSec +
          " sec/km)"
      );
      */
    },
    // return the distance input in km
    distanceKm(dist) {
      // miles
      if (dist.units[dist.unitIndex] === units.mi) {
        return dist.input * 1.609344;
      }
      // meters
      if (dist.units[dist.unitIndex] === units.m) {
        return dist.input / 1000.0;
      }
      // default: km
      return dist.input;
    },
    // return the pace input in min/km
    paceMinKm(pace) {
      // min/miles
      if (pace.units[pace.unitIndex] === units.min_mi) {
        return pace.input / 1.609344;
      }

      // default : min/km
      return pace.input;
    },
    inputToSec(input) {
      return this.countSec(this.strToTimeObj(input));
    },
    inputToFloat(input) {
      return parseFloat(input);
    },
    /* countSec : return the number of seconds in the time object */
    countSec(timeObj) {
      return timeObj.h * 3600 + timeObj.m * 60 + timeObj.s;
    },
    /* strToTimeObj: return object { h='', m='', s=''} */
    strToTimeObj(str) {
      let hour = 0;
      let min = 0;
      let sec = 0;
      // nums contains the separate digits
      let nums = [];
      let pos = str.indexOf(":");

      // loop through all numbers between ':'
      while (pos !== -1) {
        nums.push(parseInt(str.substr(0, pos)));
        str = str.substr(pos + 1, str.length - pos);

        pos = str.indexOf(":");

        // only the last digit remaining
        if (pos === -1 && (str.length === 1 || str.length === 2)) {
          nums.push(parseInt(str));
        }
      }

      // assign the parsed value to the right time digit
      if (nums.length === 2) {
        min = nums[0];
        sec = nums[1];
      } else if (nums.length === 3) {
        hour = nums[0];
        min = nums[1];
        sec = nums[2];
      }

      return {
        h: hour,
        m: min,
        s: sec
      };
    },
    // secToTimeStr: return a string formatted "mm:ss" or "hh:mm:ss" based on the number of seconds
    // source: https://gist.github.com/vankasteelj/74ab7793133f4b257ea3
    secToTimeStr(secs) {
      var pad = function(num, size) {
          return ("000" + num).slice(size * -1);
        },
        h = Math.floor(secs / 3600),
        m = Math.floor(secs / 60) % 60,
        s = Math.floor(secs - h * 3600 - m * 60);

      if (h === 0) {
        return pad(m, 2) + ":" + pad(s, 2);
      }

      return pad(h, 2) + ":" + pad(m, 2) + ":" + pad(s, 2);
    },
    // secToTimeStrWithHour: always display hours, even if h==0
    secToTimeStrWithHour(secs) {
      var pad = function(num, size) {
          return ("000" + num).slice(size * -1);
        },
        h = Math.floor(secs / 3600),
        m = Math.floor(secs / 60) % 60,
        s = Math.floor(secs - h * 3600 - m * 60);

      return pad(h, 2) + ":" + pad(m, 2) + ":" + pad(s, 2);
    },
    // triggerCalculation : calculate a parameter if the two others are valid
    triggerCalculation() {
      if (this.time.userInput && this.dist.userInput && !this.pace.userInput) {
        // try to calculate pace
        this.calculatePace();
      } else if (
        this.time.userInput &&
        !this.dist.userInput &&
        this.pace.userInput
      ) {
        // try to calculate distance
        this.calculateDistance();
      } else if (
        !this.time.userInput &&
        this.dist.userInput &&
        this.pace.userInput
      ) {
        // try to calculate time
        this.calculateTime();
      }
    },
    reset() {
      this.time.input = "";
      this.time.value = 0;
      this.time.userInput = false;
      this.time.errMessage = "";
      this.time.calculated = false;

      this.dist.value = 0;
      this.dist.input = "";
      this.dist.userInput = false;
      this.dist.errMessage = "";
      this.dist.calculated = false;

      this.pace.input = "";
      this.pace.value = 0;
      this.pace.userInput = false;
      this.pace.errMessage = "";
      this.pace.calculated = false;
    },
    // rules: source: https://stackoverflow.com/questions/8318236/regex-pattern-for-hhmmss-time-string Stephen Quan
    validateTimeStr(str) {
      // original: ^([0-1]?\d|2[0-3])(?::([0-5]?\d))?(?::([0-5]?\d))?$
      // edit: ^([0-5]?\d)(?::([0-5]?\d))?(?::([0-5]?\d))?$
      // must double the backslash...

      // at least one ':', eliminate the choice of seconds only...
      if (str.indexOf(":") === -1) {
        return false;
      }

      return RegExp("^([0-5]?\\d)(?::([0-5]?\\d))?(?::([0-5]?\\d))?$").test(
        str
      );
    },
    validateFloatStr(str) {
      return !Number.isNaN(parseFloat(str));
    }
  }
};
</script>
