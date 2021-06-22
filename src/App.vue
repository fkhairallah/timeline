<template>
  <div id="app">
    <div class="widgetWrapper" :class="{ widgetWrapperDark: darkMode }">
      <div class="pickerToolbar">
        <div class="legend mdl-grid">
          <div
            class="legendItem mdl-cell mdl-cell--1-col"
            v-for="(i, key) in switchStatesCount"
            :key="key"
            @click="setCurrentSwitchState(key)"
          >
            <div
              class="legendRec"
              :class="{ legendRecSelected: key == currSwitchstate }"
              :style="{ background: scaleValueColors[key] }"
            ></div>
            <span
              ><b>{{ switchStates[key] }}</b></span
            >
          </div>
        </div>
        <div class="legendRight">
          <div class="currentValue">
            <!--
          <div class="currentRec legendItemWrapper" :style="{background: scaleValueColors[currSwitchstate]}"></div>
          <span class="labeltLegendRight">current</span>
        -->
            <span class="labeltLegendRight" v-if="showNameItem"
              ><input class="labelInput" type="text" v-model="nameItem"
            /></span>
          </div>
          <div class="legendEvent" v-if="eventTrigger === true">
            <div class="legendItemWrapper">
              <i class="material-icons eventIcon">schedule</i>
            </div>
          </div>
        </div>
      </div>
      <div class="stateChangedHint" v-if="statesChanged == 1">
        <p>You have changed the call parameter!</p>
        <p>Switchmatrix will be ereased.</p>
        <p>Do you want to continue?</p>
        <div>
          <button
            class="
              mdl-button mdl-button--raised
              mdl-js-button mdl-js-ripple-effect
              mdl-button--accent
            "
            @click="firstInitValueArray()"
          >
            yes
          </button>
          <span></span>
          <button
            class="
              mdl-button mdl-button--raised
              mdl-js-button mdl-js-ripple-effect
              mdl-button--accent
            "
            @click="stateChangedAbort()"
          >
            no
          </button>
        </div>
      </div>
      <div class="stateChangedError" v-if="statesChanged == 2">
        <p>please contact your admin</p>
        <i class="material-icons">clear</i>
      </div>
      <div v-if="statesChanged == 0">
        <div id="contentWrapper">
          <div class="pickerContent">
            <div class="yAxis">
              <div v-for="(i, key) in yAxisLabel" :key="key">{{ i.label }}</div>
            </div>
            <div class="scaleWrapper">
              <div class="xAxis">
                <span v-for="(item, key) in currTimeScale" :key="key">{{
                  currTimeScale[key]
                }}</span>
              </div>
              <div class="scaleSet">
                <time-scale
                  v-for="(j, key) in numberScales"
                  :key="key"
                  :indexLine="j"
                  :initValue="initValue"
                  :currSwitchstate="currSwitchstate"
                  :scaleColors="scaleValueColors"
                  :eventTrigger="eventTrigger"
                  :zoomFactor="zoomFactor1"
                  :currTime="currTime1"
                ></time-scale>
              </div>
              <div class="xAxis xaBottom">
                <span v-for="(item, key) in currTimeScale" :key="key">{{
                  currTimeScale[key]
                }}</span>
              </div>
            </div>
          </div>
          <tl-inactive
            v-if="deactivation && tl_inactive"
            :lang="currLang"
            :last-state="tl_inactive_lastState"
            @click="toggleDisable"
          ></tl-inactive>
        </div>
        <div id="pickerFooter">
          <div
            class="zoomSlider"
            :class="[
              orientation == 'portrait' ? 'zoomSlider_port' : 'zoomSlider_land',
            ]"
            v-show="zoomVisible"
          >
            <div class="scaleZoom">
              <input
                id="slider1"
                class="mdl-slider mdl-js-slider"
                v-model="zoomFactor"
                type="range"
                min="0"
                max="2"
                step="1"
                tabindex="0"
              />
              <div class="xAxis_zoom">
                <span>x1</span>
                <span>x3</span>
                <span>x6</span>
              </div>
            </div>
            <div
              class="timeRange"
              v-show="!(orientation == 'portrait' && zoomFactor == 0)"
            >
              <input
                class="mdl-slider mdl-js-slider"
                v-model="currTime"
                type="range"
                min="0"
                max="24"
                step="2"
                tabindex="0"
                :disabled="zoomFactor == 0"
              />
              <div class="xAxis_zoom xAxis_firstLast">
                <span v-for="(item, key) in xAxisTimeScale" :key="key">{{
                  xAxisTimeScale[key]
                }}</span>
              </div>
            </div>
          </div>
          <div id="footerButton">
            <button
              class="
                mdl-button mdl-button--raised
                mdl-js-button mdl-js-ripple-effect
                mdl-button--accent
              "
              @click="setInterval()"
            >
              save
              <span class="mdl-button__ripple-container">
                <span class="mdl-ripple"></span>
              </span>
            </button>
            <span class="space"></span>
            <button
              class="
                mdl-button mdl-button--raised
                mdl-js-button mdl-js-ripple-effect
              "
              @click="copyMonday()"
            >
              Make all Days Like Monday
            </button>
            <span class="space"></span>
            <div
              id="deactivate"
              class="activation"
              @click="toggleDisable"
              v-if="deactivation && !secPrompt && !tl_inactive"
            >
              <div>{{ line4 }}</div>
            </div>
            <button
              id="btn-secPrompt"
              class="
                mdl-button mdl-button--raised
                mdl-js-button mdl-js-ripple-effect
                mdl-button--accent
              "
              v-if="secPrompt"
              @click="saveInterval()"
            >
              sure
              <span class="mdl-button__ripple-container">
                <span class="mdl-ripple"></span>
              </span>
            </button>
            <button
              class="
                mdl-button mdl-button--raised
                mdl-js-button mdl-js-ripple-effect
                mdl-button--accent
              "
              v-if="secPrompt"
              @click="abort()"
            >
              abort
              <span class="mdl-button__ripple-container">
                <span class="mdl-ripple"></span>
              </span>
            </button>
          </div>
          <div v-if="sideCarPresent">
            <span>Sidecar:</span>
            <span>{{ sideCar }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import tlInactive from './components/tl-inactive.vue';
import timeScale from './components/time-scale.vue';


const prefs = {
  "colorsets": {
    "1": ['#b86065', '#B5B5B5', '#3f51b5', '#515779', '#B5AB3F', '#797551', '#4A4A4A'],              // default colorset, first element is for manual mode
    "2": ['#b86065', '#d3d3d3', '#3f51b5', '#BEA472', '#E8993A', '#433B4F', '#7D3AE8'],
    "3": ['#b86065', '#E6E6E6', '#3f51b5', '#7C3FB5', '#B53F9F', '#4A4A4A', '#8C8C8C']
  },
  "languages": {
    "de": {
      "yLabel": ['Mo - So', 'Mo - Fr', 'Montag', 'Dienstag', 'Mittwoch', 'Donnerstag', 'Freitag', 'Samstag', 'Sonntag', 'Sa - So'],
      "inactive": {
        "line1": "inaktiv",
        "line2": "letzter Zustand",
        "line3": "aktivieren",
        "line4": "deaktivieren"
      }
    },
    "en": {
      "yLabel": ['mo - su', 'mo - fr', 'monday', 'tuesday', 'wednesday', 'thursday', 'friday', 'saturday', 'sunday', 'sa - su'],
      "inactive": {
        "line1": "inactive",
        "line2": "last state",
        "line3": "activate",
        "line4": "deactivate"
      }
    },
    "pt": {
      "yLabel": ['Seg - Dom', 'Seg - Sex', 'Segunda', 'Ter\u00E7a', 'Quarta', 'Quinta', 'Sexta', 'S\u00E1bado', 'Domingo', 'S\u00E1b - Dom'],
      "inactive": {
        "line1": "inactivamente",
        "line2": "\u00FAltimo estado",
        "line3": "activar",
        "line4": "desativar"
      }
    },
    "it": {
      "yLabel": ['Lun - Dom', 'Lun - Ven', 'Lunedi', 'Martedi', 'Mercoledi', 'Giovedi', 'Venerdi', 'Sabato', 'Domenica', 'Sab-Dom'],
      "inactive": {
        "line1": "in modo inattivo",
        "line2": "ultimo stato",
        "line3": "attivarsi",
        "line4": "disattivarsi"
      }
    },
    "nl": {
      "yLabel": ['ma - zo', 'ma - vr', 'maandag', 'dinsdag', 'woensdag', 'donderdag', 'vrijdag', 'zaterdag', 'zondag', 'za - zo'],
      "inactive": {
        "line1": "inactief",
        "line2": "vorige staat",
        "line3": "activeren",
        "line4": "deactiveren"
      }
    }
  }
}


export default {
  name: "App",
  components: {
    tlInactive,
    timeScale,
  },
    data: () => ({
    device: '',               // variables for presentation zoom slider and slider for shift timeintervall in sitemap
    orientation: 'portrait',  //
    zoomVisible: false,       //
    zoomForced: 'auto',       // 
    zoomFactor: 0,            //
    currTime: 12,            //
    initValue: {},            // buffer for data structure
    numberScales: 0,
    yAxisLabel: [],
    timeScale: ['00', '01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'],
    urlParamItemName: '',
    secPrompt: false,         // security prompt befor switch points will changed
    ip: '',
    switchStates: [],
    sideCarPresent: false,
    sideCar: '',
    currSwitchstate: 0,
    scaleValueColors: [],
    stateParam: ',',
    statesChanged: 0,         // 0 .. not changed, 1 .. changed, 2 .. error
    apiRequestBody: '',
    eventTrigger: false,
    darkMode: false,
    currLang: 'en',           // default language: en
    line4: '',
    tl_inactive: false,       // timeline inactive = true
    tl_inactive_lastState: '',// state befor timeline inactivated
    deactivation: false,      // switch for activation/deactivation visible
    debounceTimer: null,       // debonce timer for activation/deactivation
    showNameItem: false,      // show the name of the timeline
    urlParamLabelName: null,   // nameitem to retrieve from OpenHab
    nameItem: null,           // name of timeline

  }),
  computed: {
    zoomFactor1: function () {
      return (this.zoomFactor == 0) ? 1 : this.zoomFactor * 3
    },
    currTime1: function () {
      return this.currTime * 1
    },
    currTimeScale: function () {
      let startIndex = this.currTime - (24 / this.zoomFactor1 / 2)
      if (startIndex < 0) startIndex = 0
      if ((startIndex + (24 / this.zoomFactor1)) > 24) startIndex = (24 - (24 / this.zoomFactor1))
      let endIndex = startIndex + (24 / this.zoomFactor1)
      return this.timeScale.filter((val, index) => ((this.zoomFactor1 == 1) && (index % 2 == 0)) || ((this.zoomFactor1 != 1) && ((index >= startIndex) && (index <= endIndex))))
    },
    xAxisTimeScale: function () {
      let calculatedLabel = []
      if ((this.device == 'Mobile') && (this.orientation == 'portrait')) {
        calculatedLabel = this.timeScale.filter((val, index) => (index % 4 == 0))
      } else {
        calculatedLabel = this.timeScale.filter((val, index) => (index % 2 == 0))
      }
      return calculatedLabel;
    }
  },
  created() {
    console.log("Created!");
    let urlParams = new URLSearchParams(window.location.search);
    this.urlParamItemName = urlParams.get('transferItem');
    this.urlParamLabelName = urlParams.get('nameitem');
    let yAxis = (urlParams.get('yAxisLabel')).split(',');
    let openHAB_ip = urlParams.has('ip') ? urlParams.get('ip') : location.host;
    this.stateParam = urlParams.get('states');
    let colorSet = urlParams.get('colorset');
    this.deactivation = ((urlParams.get('deactivation') != null) && (urlParams.get('deactivation') == 'true')) ? true : false;

    // set language then selected else default: en
    if (urlParams.get('lang') != null) this.currLang = urlParams.get('lang');
    this.currLang = (prefs.languages[this.currLang] == undefined) ? 'en' : this.currLang;
    // select correct language for UI
    this.line4 = prefs.languages[this.currLang].inactive.line4;

    // set default for states values OFF/ON
    if (this.stateParam == null) this.stateParam = 'OFF,ON';
    this.switchStates = this.stateParam.split(',');
    // when more then five states passed -> cancel from position six
    if (this.switchStates.length > 5) this.switchStates.splice(6, this.switchStates.length - 6);

    // check user defined colorset or select one of the predefined colorsets or set user defined colors
    // select default colorset
    this.scaleValueColors = prefs.colorsets["1"]

    // check if url parameter used
    if (colorSet != null) {
      let csArray = colorSet.split(',');
      // check if selected an predefined colorset (lenght = 1) or userdefined colorset (lenght > 1)
      if (csArray.length > 1) {
        // check is event inactive then set #000 as dummy
        if (urlParams.get('event') != 'yes') {
          csArray.unshift('#000')
        }
        csArray.forEach((color, index) => {
          if (index < this.scaleValueColors.length) {
            this.scaleValueColors[index] = '#' + color;
          }
        })
      } else {
        // check is selected colorset not empty and exist key in prefs
        if (csArray[0] != "" && (csArray[0] in prefs.colorsets)) {
          this.scaleValueColors = prefs.colorsets[csArray[0]]
        }
      }
    }

    // show event trigger
    if (urlParams.get('event') == 'yes') {
      this.eventTrigger = true;
      // add N/A (-1) as state; note: there are max 5 user defined states
      this.switchStates.unshift('N/A');
      this.currSwitchstate = 1;   // switch to the first real state 
    } else {
      this.scaleValueColors.shift();                    // event mode no -> remove the predefined color for manual mode
    }

    // check dark mode
    if (urlParams.get('dark') == 'yes') this.darkMode = true;
    this.switchStatesCount = this.switchStates.length;
    this.ip = 'http://' + openHAB_ip + '/rest/items/';
    // check zoom parameter
    let zoom = urlParams.get('zoom');
    if (zoom == 'no') this.zoomForced = 'no';
    if (zoom == 'force') this.zoomForced = 'force';



    // set label for y axis and note the language select; default is german
    let yLabel = this._selectLang(this.currLang);
    if (yAxis.includes('17')) this.yAxisLabel.push({ 'key': '17', 'label': yLabel[0] });
    if (yAxis.includes('15')) this.yAxisLabel.push({ 'key': '15', 'label': yLabel[1] });
    if (yAxis.includes('1')) this.yAxisLabel.push({ 'key': '1', 'label': yLabel[2] });
    if (yAxis.includes('2')) this.yAxisLabel.push({ 'key': '2', 'label': yLabel[3] });
    if (yAxis.includes('3')) this.yAxisLabel.push({ 'key': '3', 'label': yLabel[4] });
    if (yAxis.includes('4')) this.yAxisLabel.push({ 'key': '4', 'label': yLabel[5] });
    if (yAxis.includes('5')) this.yAxisLabel.push({ 'key': '5', 'label': yLabel[6] });
    if (yAxis.includes('6')) this.yAxisLabel.push({ 'key': '6', 'label': yLabel[7] });
    if (yAxis.includes('7')) this.yAxisLabel.push({ 'key': '7', 'label': yLabel[8] });
    if (yAxis.includes('67')) this.yAxisLabel.push({ 'key': '67', 'label': yLabel[9] });
    this.numberScales = this.yAxisLabel.length;

    // init for initValue, because the delay from promise throws error while rendering
    for (let i = 1; i < (this.numberScales + 1); i++) {
      //this.$set(this.initValue, i, { 'key': this.yAxisLabel[i - 1].key, 'value': this._initArray(96) });
      this.initValue[i] =  { 'key': this.yAxisLabel[i - 1].key, 'value': this._initArray(96) };
    }

    //this.$set(this.initValue, 99, this.switchStates.toString())
    this.initValue[99] = this.switchStates.toString();
    //this.$set(this.initValue, 100, {
    this.initValue[100] = {
      'event': this.eventTrigger,
      'lastItemState': (-1),
      'inactive': false
    };

    // retrieve the name of the timeline
    if (this.urlParamLabelName) {
      console.log("Retrieving ", this.urlParamLabelName);
      fetch(this.ip + this.urlParamLabelName + '/state').then(response => {
        if (response.status === 200) {
          console.log("Got " + response.data);
          this.nameItem = response.data;
          this.showNameItem = true;
        }
        else {
          console.log('warn in API call; transfered data is empty or not valid', response.status);
        }
      }, response => {
        // error callback
        console.log('error in nameItem API request to openHAB')
      });
    }

    console.log(`request=${this.ip}${this.urlParamItemName}/state`);
    // retrieve the transferItem object
    fetch(this.ip + this.urlParamItemName + '/state').then(response => {
      this.apiRequestBody = response.body;
      if ((typeof this.apiRequestBody === 'object') && (response.status === 200)) {
        // check inital call
        // check integrity switchStates
        if (this.apiRequestBody[99] !== undefined) {
          if (this.apiRequestBody[99] !== null) {
            let storedStates = this.apiRequestBody[99].split(',');

            if (storedStates.length == this.switchStatesCount) {
              for (let i = 0; i < this.switchStatesCount; i++) {
                if (this.switchStates[i] != storedStates[i]) this.statesChanged = 1;
              };
            } else {
              this.statesChanged = 1;
            }
          } else {
            if (this.stateParam !== null) {
              if (!((this.stateParam.includes('ON')) && (this.stateParam.includes('OFF')))) {
                this.statesChanged = 1;
              }
            }
          }
        }
        if (this.apiRequestBody[100] !== null) {
          if (this.apiRequestBody[100].event != this.eventTrigger) this.statesChanged = 1;                                    // check integrity of event parameter   
          if (this.apiRequestBody[100].inactive == true) {                                                                    // check state of inactive
            this.tl_inactive = true;
            if (this.apiRequestBody[100].lastState != '') this.tl_inactive_lastState = this.apiRequestBody[100].lastState;    // state befor timeline inactivated
          }
        }
        // check integrity of y axis parameter
        let requestKeys = Object.keys(this.apiRequestBody)
        let daySet = 0;
        let dayKeys = [];
        for (let i = 0; i < requestKeys.length; i++) {
          if (this.apiRequestBody[requestKeys[i]].key != undefined) {
            dayKeys.push(this.apiRequestBody[requestKeys[i]].key);
            daySet = ++daySet;
          }
        }
        if (yAxis.length = daySet) {
          for (let i = 0; i < daySet; i++) {
            if (!dayKeys.includes(yAxis[i])) {
              this.statesChanged = 1;
              break;
            };
          }
        } else {
          this.statesChanged = 1;
        }


        if (this.statesChanged == 0) {
          this._initValueArray();
        }
      } else {
        console.log('warn in API call; transfered data is empty or not valid')
      }
      
    }, response => {
      // error callback
      console.log('error in API request to openHAB')
    });
    console.log("done created");

  },
  mounted() {
    this.$nextTick(function () {
      window.addEventListener('resize', this.screenChanged);
      //Init
      this.screenChanged();
      
      // now load the sidecar info
      this.sideCarPresent = true;
      this.sideCar = this.apiRequestBody.toString();


    })
        console.log("done mounted");

  },
  async beforeUnmount() {
    console.log('before unmounting');
  },

  beforeDestroy() {
    window.removeEventListener('resize', this.screenChanged);
  },
  methods: {

    _initArray(n) {
      var arr = [];
      let defaultState = (this.eventTrigger) ? (-1) : 0;
      for (let i = 0; i < n; i++) arr.push(defaultState);
      return arr;
    },
    _initValueArray() {
      for (let i = 1; i < (this.numberScales + 1); i++) {
        // init data set
        this.initValue[i] = {
          'key': this.yAxisLabel[i - 1].key,
          'value': (typeof this.apiRequestBody[i] !== 'undefined') ? this.apiRequestBody[i].value : this._initArray(96)
        };
      }
    },
    firstInitValueArray() {
      this.statesChanged = 0;
    },
    stateChangedAbort() {
      this.statesChanged = 2;
    },
    // set color is active
    setCurrentSwitchState(key) {
      this.currSwitchstate = key;
    },
    setInterval() {
      this.secPrompt = true;
    },
    copyMonday() { // copy Monday's data into the rest of the week
      console.log('All days same as monday');
      for(var i=2; i<8;i++) // Tuesday - Sunday
        this.initValue[i].value.forEach((v,j)=> {
          if (this.initValue[i].value[j] != this.initValue[1].value[j]) {
            this.initValue[i].value[j] = this.initValue[1].value[j]
            this.initValue[i]['value'][j] = this.initValue[1].value[j];
          }
        });
      
    },
    saveInterval() {

      // save the transfer item
      this.$http.post(this.ip + this.urlParamItemName, JSON.stringify(this.initValue), { 'headers': { "Accept": "application/json", "content-type": "text/plain" } }).then(response => {
        // get status
        console.log(response.status);
        // get status text
        console.log(response.statusText);
      }, response => {
        // error callback
        console.log(response)
      });

      // and the name item
      this.$http.post(this.ip + this.urlParamLabelName, this.nameItem, { 'headers': { "Accept": "application/json", "content-type": "text/plain" } }).then(response => {
        // get status
        console.log(response.status);
        // get status text
        console.log(response.statusText);
      }, response => {
        // error callback
        console.log(response)
      });
      this.secPrompt = false;
    },
    abort() {
      this.secPrompt = false;
    },
    toggleDisable() {
      this.tl_inactive = (this.tl_inactive == true) ? false : true;
      if (this.debounceTimer == null) {
        this.debounceTimer = setTimeout(this._debounceTimerFunc, 2000, this);
      } else {
        clearTimeout(this.debounceTimer);
        this.debounceTimer = setTimeout(this._debounceTimerFunc, 2000, this);
      }
    },
    _debounceTimerFunc(oThis) {
      if (oThis.tl_inactive != oThis.initValue["100"].inactive) {
        oThis.initValue["100"].inactive = oThis.tl_inactive;
        this.saveInterval();
        if (oThis.tl_inactive) {
          this.$http.get(oThis.ip + 'TimelineHelper/state').then(response => {
            if ((typeof response.body === 'object') && (response.status === 200)) {
              if (response.body[oThis.urlParamItemName] != undefined) {
                if (response.body[oThis.urlParamItemName]["inactiveLastValue"] != undefined)
                  oThis.tl_inactive_lastState = response.body[oThis.urlParamItemName]["inactiveLastValue"];
              };
            } else {
              console.log('item TimelineHelper, data strucktur is not valid')
            }
          }, response => {
            // error callback
            console.log('error in API request to openHAB or item TimelineHelper is not defined or empty')
          });
        }
      }
      oThis.debounceTimer = null;
    },
    _selectLang(lang) {
      return (prefs.languages[lang] == undefined) ? prefs.languages["en"].yLabel : prefs.languages[lang].yLabel
    },
    screenChanged(event) {
      //this.screenWidth = document.documentElement.clientWidth;
      //this.screen Height = document.documentElement.clientWidth;

      // recognize device type
      if (navigator.userAgent.match(/mobile/i)) {
        this.device = 'Mobile';
      } else if (navigator.userAgent.match(/iPad|Android|Touch/i)) {
        this.device = 'Tablet';
      } else {
        this.device = 'Desktop';
      }
      // set zoom variables for presentation in sitemap
      this.zoomVisible = ((this.device != 'Desktop') && (this.zoomForced != 'no')) ? true : false;
      if (this.zoomForced == 'force') this.zoomVisible = true;

      // recognize device orientation
      if (window.matchMedia("(orientation: portrait)").matches) this.orientation = 'portrait'
      if (window.matchMedia("(orientation: landscape)").matches) this.orientation = 'landscape'
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
