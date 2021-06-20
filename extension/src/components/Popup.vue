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
    class="overflow-hidden overscroll-none p-1 bg-dark text-blue px-2 py-1"
    style="min-height: 75px; width: 800px; height: 600px;"
  >
    <div class="flex">
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
      <!-- Meta buttons always displayed with GitHub repos link and settings -->
      <div class="flex justify-end">
        <img
          src="icons/github.svg"
          class="w-7 hover:bg-lightblue bg-lightbluelighted rounded p-1"
          alt="code icon"
          title="Checkout the code on GitHub, it's opensource !"
          @click="goToSourceCode"
        />
        <img
          src="icons/settings.svg"
          alt="settings icon"
          title="Open/Close settings"
          class="w-7 hover:bg-lightblue bg-lightbluelighted rounded px-1 ml-1"
          :class="{ 'bg-blue-400': settingsEnabled }"
          @click="openSettings"
        />
      </div>
    </div>

    <hr class="border-blue my-1" />

    <!-- Extension body -->

    <!-- Messages about errors, loading and stid -->
    <div v-if="settingsEnabled">
      stid
      <input
        class="w-16 px-1"
        type="number"
        placeholder="default"
        v-model="config.stid"
        @change="console.log(config.stid)"
      />
    </div>
    <div v-if="notLogged" class="message">
      Vous n'êtes pas connecté à l'intranet du CPNV... Essayez à nouveau une
      fois connecté.
    </div>
    <div v-if="noInternshipFound" class="message">
      Il n'y a pas de stage avec l'identifiant {{ config.stid }}...
    </div>
    <div v-if="loading" class="message">
      Chargement des données...
    </div>

    <!-- Stats -->
    <div v-if="displayInfo">
      <div
        style="height: 92px"
        :class="{ hidden: config.stid != null && config.stid != '' }"
      >
        <span class="text-xs">Temps total travaillé / Temps total requis</span>
        <div class="text-sm">
          <span :title="'Précisément ' + totaltime">
            {{ roundWith2Decimals(totaltime) }} </span
          >/
          <span :title="'Précisément ' + totaltimetowork"
            >{{ roundWith2Decimals(totaltimetowork) }} h.</span
          >
        </div>
        <div>
          <span class="text-xs">Temps moyen (par jour): </span>
          <span class="text-sm">
            <span :title="'Précisément ' + timeperday">{{
              roundWith2Decimals(timeperday)
            }}</span
            >h/{{ 8.2 }}h
          </span>
        </div>
        <div>
          <span class="text-xs">Temps en retard:</span>
          <span class="text-sm">
            {{ diffInHours }} h. = {{ diffInDays }} jours
          </span>
        </div>
        <!-- {{ this.entries.length }} -->
      </div>
      <div
        class="message"
        :class="{ hidden: config.stid == null || config.stid == '' }"
      >
        Statistiques désactivées.
      </div>
    </div>

    <hr class="border-blue my-1" v-if="displayInfo" />

    <!-- Logbook content -->
    <div class="w-full" v-if="displayInfo">
      <div class="flex border-blue border-b pb-1 mb-1">
        <span class="flex-1"
          >Journal de stage de
          {{ internship.student != null ? internship.student : "?" }} chez
          {{ internship.company != null ? internship.company : "?" }}</span
        ><span class="text-sm">
          <span class="text-red-300 text-xs">{{ mention }}</span>
          <select class="bg-blue" v-model="nbDaysToDisplay">
            <option value="1">1 jour</option>
            <option value="2">2 jours</option>
            <option value="7">1 semaine</option>
            <option value="31">1 mois</option>
            <option value="-1">Tout</option>
          </select>
        </span>
      </div>
      <div
        class="text-xs overflow-y-scroll overflow-x-hidden break-normal"
        style="max-height: 390px; max-width: 900px;"
      >
        <div
          class="mt-1 mb-2 pb-1"
          v-for="(entriesOfADay, index) in entriesToDisplay"
          :key="index"
        >
          <span class="font-extrabold text-sm"
            >Le {{ formatDate(entriesOfADay[0].date) }}</span
          >:
          <div v-for="(entry, index) in entriesOfADay" :key="index">
            <div class="font-bold">· {{ entry.time }}h</div>
            <div
              class="italic break-normal ml-3 pl-1 mr-2 whitespace-pre-line border-blue border-l mb-1"
            >
              {{ entry.text }}
            </div>
          </div>
          <hr class="border-blue mr-3 mt-2" />
        </div>
        <div class="italic text-xs" v-if="nbDaysToDisplay != null">
          Jours suivants masqués...
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

const startTime =
  "<input type='number' name='duration' step='0.25' max='12' min='0.25' value='";
const endTime = "'>";
const startDescription = "colspan=2 style='display: none;'>";
const endDescription = "</td>";
const startDate = "<input type='hidden' name='date' value='";
const endDate = "'>";
const startStudent =
  "<input id='expand' type='button' value='Dérouler' style='float:right'/><h2>Stage de ";
const endStudent = ", ";
const startCompany = ", ";
const endCompany = "</h2>";

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
      config: {
        stid: null, //"stage id" = internship id, if null it's the current internship of the logged person
        savedStids: [], //several saved stid value (id and name)
        keywordsOfEntryToIgnore: [] //keywords present in the description of the entries to ignore
      },
      mention: "",
      nbDaysToDisplay: 2,
      loading: true,
      notLogged: false,
      noInternshipFound: false,
      settingsEnabled: false,
      entries: null,
      internship: {
        student: "",
        company: ""
      },
      pageRawContent: null,
      totaltime: null,
      timeperday: null,
      totaltimetowork: null,
      diffInHours: null,
      diffInDays: null
    };
  },
  computed: {
    //Display logbook information only if data are present
    displayInfo() {
      return !this.settingsEnabled && !this.loading && !this.noInternshipFound;
    },
    entriesToDisplay() {
      console.log(this.nbDaysToDisplay);
      console.log(this.entries);
      console.log(this.entriesByDay);
      if (this.entries != null) {
        if (this.nbDaysToDisplay == "-1") {
          //Option "All" is selected
          return this.entriesByDay; //return all entries without any filter
        }
        return this.entriesByDay.filter((value, index) => {
          return index + 1 <= this.nbDaysToDisplay;
        });
      }
      return [];
    },
    entriesByDay() {
      var array = [];
      var dayCount = -1;
      var lastDate = null;
      if (this.entries != null) {
        console.log("debug entries by day");
        for (var entry of this.entries) {
          if (lastDate != entry.date) {
            dayCount++;
            lastDate = entry.date;
          }
          if (array[dayCount] == undefined) {
            array[dayCount] = [];
          }
          array[dayCount].push(entry);
        }
        console.log(array);
        return array;
      }
      return [];
    }
  },
  methods: {
    //Format a date given in Y-m-d format in the d.m.Y format
    formatDate(value) {
      console.log(value);
      return (
        value.substr(8) + "." + value.substr(5, 2) + "." + value.substr(0, 4)
      );
    },
    //Go to source code button (icon with anchor) on GitHub
    goToSourceCode() {
      window.open("https://github.com/samuelroland/logbookHelper", "_blank");
    },
    //Open settings and focus on create link input
    openSettings() {
      this.settingsEnabled = !this.settingsEnabled; //invert settings status

      //if settings have been closed
      if (this.settingsEnabled == false) {
        this.loadLogbookData();
      }
    },
    loadLogbookData() {
      this.loading = true;
      axios.defaults.timeout = 1000;

      axios
        .post(
          "https://intranet.cpnv.ch/stages/Journal.php",
          this.config.stid != null && this.config.stid != ""
            ? "stid=" + this.config.stid
            : null
        )
        .then(response => {
          console.log("data is here !");

          if (
            response.data.indexOf(
              "Vous devez vous authentifier pour accéder à cette page"
            ) != -1
          ) {
            this.notLogged = true;
            this.extractLocalDataIfExist();
            this.mention = "Données locales chargées...";
          } else if (response.data.indexOf("<h2>Stage de  , </h2>") != -1) {
            this.noInternshipFound = true;
          } else {
            this.pageRawContent = response.data;
            this.extractLogbookData();
          }
          this.loading = false;
        })
        .catch(e => {
          console.log(e);
          this.loading = false;
          if (this.extractLocalDataIfExist()) {
            this.mention = "Données locales. Connexion échouée.";
          } else {
            this.mention = "Pas de données. Connexion échouée.";
          }
        });
    },
    extractLocalDataIfExist() {
      browser.storage.local.get().then(data => {
        if (data.entries != null) {
          console.log("local data found !");
          this.entries = data.entries;
          this.internship = data.internship;
          return true;
        }
        console.log("local data not found !");
        return false;
      });
    },
    extractLogbookData() {
      console.log("extractLogbookData launched");
      var html = this.pageRawContent;
      console.log(html);

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
      var currentItemIgnored; //is the current item ignored ?

      while (html.indexOf(startDescription, currentStartPosDesc) != -1) {
        currentItemIgnored = false;
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
            html.indexOf(startTime, pos1 - (1400 + startTime.length)) +
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
            currentItemIgnored = true;
            ignoredDays.push({ text: desc, time: hours, date: date });
          }

          logIt(
            "date: " +
              date +
              " time: " +
              hours +
              "h. -" +
              (currentItemIgnored ? " day ignored" : " accepted")
          );

          count++;
        }
      }
      timeperday = totaltime / nbDays;
      this.timeperday = timeperday;
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
      console.log(count);
      this.entries = array;

      var internship = {};
      //Student fullname extraction
      var posStudent = html.indexOf(startStudent) + startStudent.length;
      internship.student = html.substr(
        posStudent,
        html.indexOf(endStudent, posStudent) - posStudent
      );

      //Company name extraction
      var posCompany =
        html.indexOf(startCompany, posStudent) + startCompany.length;
      internship.company = html.substr(
        posCompany,
        html.indexOf(endCompany) - posCompany
      );

      this.internship = internship;
      browser.storage.local.set(
        JSON.parse(
          JSON.stringify({ entries: this.entries, internship: this.internship })
        )
      );

      browser.storage.local.get().then(a => {
        console.log(a);
      });
    },
    roundWith2Decimals(value) {
      return Math.round(value * 100) / 100;
    }
  },
  mounted() {
    this.loadLogbookData();
  }
};
//Just log text in the console
function logIt(text) {
  console.log(text);
}
</script>
