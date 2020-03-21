<template>
  <v-container>
    <v-row class="text-center">
      <v-spacer></v-spacer>
      <v-col cols="4">
        <v-card>
          <v-card-title>
            <h3>Calculate your pace</h3>
          </v-card-title>
          <v-card-text>
            <v-form>
              <v-text-field
                type="text"
                label="Time"
                v-model="time.input"
                autocomplete="off"
                prepend-icon="mdi-calculator"
                :append-outer-icon="timeOuterIcon"
                :error-messages="time.errMessage"
                @change="eventTimeChanged"
                @click:prepend="calculateTime"
                @input="eventTimeInput"
                placeholder="00:00:00"
              />
              <v-menu offset-x>
                <template v-slot:activator="{ on }">
                  <v-text-field
                    type="text"
                    label="Distance"
                    v-model="dist.input"
                    autocomplete="off"
                    prepend-icon="mdi-calculator"
                    :suffix="distanceUnit"
                    :error-messages="dist.errMessage"
                    append-icon="mdi-unfold-more-horizontal"
                    :append-outer-icon="distanceOuterIcon"
                    @change="eventDistanceChanged"
                    @click:prepend="calculateDistance"
                    @click:append="toggleDistanceUnits"
                    @click:append-outer="on.click"
                    @input="eventDistanceInput"
                  />
                </template>
                <v-list dense nav>
                  <v-subheader>DISTANCES</v-subheader>
                  <v-list-item
                    v-for="(item, index) in presetDistances"
                    :key="index"
                    @click="clickPresentDistance(index)"
                  >
                    <v-list-item-title>{{ item.title }}</v-list-item-title>
                  </v-list-item>
                </v-list>
              </v-menu>
              <v-text-field
                type="text"
                label="Pace"
                v-model="pace.input"
                autocomplete="off"
                persistent-hint
                prepend-icon="mdi-calculator"
                :suffix="paceUnit"
                :error-messages="pace.errMessage"
                append-icon="mdi-unfold-more-horizontal"
                :append-outer-icon="paceOuterIcon"
                @change="eventPaceChanged"
                @click:prepend="calculatePace"
                @click:append="togglePaceUnits"
                @input="eventPaceInput"
              />
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn rounded @click="reset">=</v-btn>
            <v-btn rounded @click="reset">Reset</v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
      <v-spacer></v-spacer>
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
      errMessage: ""
    },
    dist: {
      validated: false,
      value: 0,
      units: [units.km, units.mi, units.m],
      unitIndex: 0,
      input: "",
      errMessage: ""
    },
    pace: {
      validated: false,
      units: [units.min_km, units.min_mi],
      unitIndex: 0,
      value: 0,
      input: "",
      errMessage: ""
    }
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
    clickPresentDistance(index) {
      // set the distance value
      this.dist.input = this.presetDistances[index].value;
      // set the units
      if (
        this.presetDistances[index].unit !==
        this.dist.units[this.dist.unitIndex]
      ) {
        // change the units...
        for (let i = 0; i < this.dist.units.length; i++) {
          if (this.presetDistances[index].unit === this.dist.units[i]) {
            this.dist.unitIndex = i;
            break;
          }
        }
      }

      this.clearErrorMessage(this.dist);
    },
    clearErrorMessage(obj) {
      obj.errMessage = "";
    },

    eventTimeChanged() {
      console.log("time changed! " + this.time.input);
      let secs = this.countSec(this.strToTimeObj(this.time.input));
      console.log(secs + " seconds");
      console.log(this.secToTimeStr(secs));

      this.time.validated = this.validateTimeStr(this.time.input);
    },
    eventDistanceChanged() {
      console.log("dist changed! " + this.dist.input);

      let p =
        this.countSec(this.strToTimeObj(this.time.input)) / this.dist.input;
      console.log("pace: " + p + " seconds");

      this.dist.validated = this.validateFloatStr(this.dist.input);
    },
    eventPaceChanged() {
      console.log("pace changed! " + this.pace.input);

      this.pace.validated = this.validateTimeStr(this.pace.input);
    },
    eventTimeInput(input) {
      console.log("time input: " + input);
      // clear the message error if the value is valid
      if (this.time.errMessage !== "") {
        if (this.validateTimeStr(input)) {
          this.clearErrorMessage(this.time);
        }
      }
    },
    eventDistanceInput(input) {
      console.log("distance input: " + input);

      // clear the message error if the value is valid
      if (this.dist.errMessage !== "") {
        if (this.validateFloatStr(input)) {
          this.clearErrorMessage(this.dist);
        }
      }
    },
    eventPaceInput(input) {
      console.log("pace input: " + input);

      // clear the message error if the value is valid
      if (this.pace.errMessage !== "") {
        if (this.validateTimeStr(input)) {
          this.clearErrorMessage(this.pace);
        }
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

      let timeSec = paceSec * distKm;

      this.time.input = this.secToTimeStr(timeSec);

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
      );
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

      let distKm = timeSec / paceSec;

      // convert to used units
      if (this.dist.units[this.dist.unitIndex] === units.m) {
        distKm *= 1000.0;
      } else if (this.dist.units[this.dist.unitIndex] === units.mi) {
        distKm /= 1.609344;
      }

      this.dist.input = distKm.toFixed(1);

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

      let paceSec = Math.floor(timeSec / distKm);

      if (this.pace.units[this.pace.unitIndex] === units.min_mi) {
        paceSec *= 1.609344;
      }

      this.pace.input = this.secToTimeStr(paceSec);

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
    reset() {
      this.time.input = "";
      this.time.value = 0;
      this.time.validated = false;
      this.time.errMessage = "";

      this.dist.value = 0;
      this.dist.input = "";
      this.dist.validated = false;
      this.dist.errMessage = "";

      this.pace.input = "";
      this.pace.value = 0;
      this.pace.validated = false;
      this.pace.errMessage = "";
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
