<!--
    logbookHelper - Internship logbook helper for CPNV students
    Copyright (C) 2021 Samuel Roland

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <https://www.gnu.org/licenses/>.
-->
<template>
  <div
    class="w-80 overflow-hidden overscroll-none p-1"
    style="min-height: 75px; color: #76bfff; background-color: #0b0e44;"
  >
    <div class="flex flex-1 min-w-max">
      <h3
        class="flex flex-row items-center m-0 text-lg cursor-help"
        title="Internship logbook helper for CPNV students"
      >
        logbookHelper
      </h3>
      <span
        class="text-xs cursor-help italic flex items-end ml-1 w-10"
        :title="'Released the ' + versionDate + '.'"
        >{{ version }}</span
      >
    </div>

    <!-- Extension body -->
    <div v-if="settingsEnabled">settings...</div>
    <div v-else>
      aaaa
      <br />
      {{ totaltime }} / {{ totaltimetowork }}<br />
      {{ diffInDays }} hours = {{ diffInDays }} days
      <!-- {{ this.entries.length }} -->
      <!-- <span v-for="entry in entries" :key="entry.text">{{ entry.text }}</span> -->
    </div>

    <!-- Footer with 2 icons: link to source code and settings button - Always displayed -->
    <div class="flex flex-1 mt-2 justify-end">
      <img
        src="icons/github.svg"
        class="w-7 hover:bg-blue-500 rounded hover:text-white p-1"
        alt="code icon"
        title="Checkout the code on GitHub, it's opensource !"
        @click="goToSourceCode"
      />
      <img
        src="icons/settings.svg"
        alt="settings icon"
        title="Open/Close settings"
        class="w-7 hover:bg-blue-500 rounded hover:text-white px-1"
        :class="{ 'bg-blue-400': settingsEnabled }"
        @click="openSettings"
      />
    </div>
  </div>
</template>

<script>
import axios from "axios";

const startTime =
  "<input type='number' name='duration' step='0.25' max='12' min='0.25' value=";
const endTime = "'>";
const startDescription = "colspan=2 style='display: none;'>";
const endDescription = "</td>";
const startDate = "<input type='hidden' name='date' value='";
const endDate = "'>";
const leaveDates = [
  "2021-05-21",
  "2021-05-24",
  "2021-05-25",
  "2021-05-26",
  "2021-05-27",
  "2021-05-28",
  "2021-04-05",
  "2021-04-02"
];

export default {
  name: "Popup",
  props: {
    //Version information
    version: String,
    versionDate: String
  },
  data() {
    return {
      settingsEnabled: false,
      entries: null,
      pageRawContent: null,
      totaltime: null,
      totaltimetowork: null,
      diffInHours: null,
      diffInDays: null
    };
  },
  computed: {},
  methods: {
    //Go to source code button (icon with anchor) on GitHub
    goToSourceCode() {
      window.open("https://github.com/samuelroland/logbookHelper", "_blank");
    },
    //Open settings and focus on create link input
    openSettings() {
      this.settingsEnabled = !this.settingsEnabled; //invert settings status
    },
    extractLogbookData() {
      console.log("extractLogbookData launched");
      var html = this.pageRawContent;

      var ignoredDays = [];

      var currentStartPosDesc = 0;
      var pos1 = 0;
      var pos2 = 0;
      var pos3 = 0;
      var desc = "";
      var date = null;
      var hours = null;
      var missingHours = null;
      var lastDate = null;

      var count = 0;
      var totaltime = 0;
      var timeperday = 0;
      var array = [];
      var nbDays = 0;
      while (html.indexOf(startDescription, currentStartPosDesc) != -1) {
        console.log(html.indexOf(startDescription, currentStartPosDesc));
        if (html.indexOf(startDescription, currentStartPosDesc) != -1) {
          pos1 =
            html.indexOf(startDescription, currentStartPosDesc) +
            startDescription.length;
          console.log(pos1);
          //logIt(pos1)
          desc = html.substr(
            pos1,
            html.indexOf(endDescription, pos1 - 1) - pos1
          );
          currentStartPosDesc = pos1 + 1;

          pos2 =
            html.indexOf(startTime, pos1 - (200 + startTime.length)) +
            startTime.length;
          //logIt(pos2)
          hours = html.substr(pos2, html.indexOf(endTime, pos2) - pos2);
          //console.log(hours)

          pos3 = html.indexOf(startDate, pos1 - 600) + startDate.length;
          //logIt(pos3)
          date = html.substr(pos3, html.indexOf(endDate, pos3) - pos3 - 9);
          //console.log(date)

          if (
            leaveDates.indexOf(date) == -1 &&
            desc.indexOf("Cours matu") == -1 &&
            desc.indexOf("Matu.") == -1 &&
            desc.indexOf("A remplir") == -1 &&
            desc != "" &&
            date.length <= 10
          ) {
            totaltime += parseFloat(hours);
            if (lastDate != date) {
              console.log("new day: " + date);
              nbDays++;
            }
            lastDate = date;
            array.push({ text: desc, time: hours, date: date });
          } else {
            ignoredDays.push({ text: desc, time: hours, date: date });
          }

          logIt("date: " + date + " time: " + hours);

          //logIt(timeperweek)
          console.log(count + " " + currentStartPosDesc + " " + hours);
          count++;
        }

        timeperday = totaltime / nbDays;
        logIt(array);
        logIt(totaltime + "/" + nbDays * 8.2);
        this.totaltime = totaltime;
        this.totaltimetowork = nbDays * 8.2;
        logIt(timeperday);
        missingHours = (8.2 - timeperday) * nbDays;
        this.diffInHours = missingHours;
        logIt(missingHours);
        logIt(missingHours / 8);
        this.diffInDays = missingHours / 8.2;

        this.entries = array;
      }
    }
  },
  mounted() {
    axios.get("http://intranet.cpnv.ch/stages/Journal.php").then(response => {
      console.log("data is here !");
      this.pageRawContent = response.data;
      this.extractLogbookData();
    });
  }
};
//Just log text in the console
function logIt(text) {
  console.log(text);
}
</script>
