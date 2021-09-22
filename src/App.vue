<template>
  <div>
    <h1>Annotate Audio Waveforms</h1>
    <p>with Vue.js + Peaks.js</p>

    <main>
<!--      <waveform ref="waveform" dataFile="test.dat" mainTrack="test.mp3" :transitionTracks="['transitions/Jingle - labo Vestaradio.mp3', 'transitions/jingle vestaradio.mp3']" />-->

            <waveform ref="waveform" :mainTrack="dataAudio" />

<!--      <ul class="playback">-->
<!--        <li><button @click="play('introTrack')" :disabled="!introTrack">Play intro</button></li>-->
<!--        <li><button @click="play('mainTrack')"><span aria-hidden>▶</span> Play</button></li>-->
<!--        <li><button @click="play('extroTrack')" :disabled="!extroTrack">Play extro/mix</button></li>-->
<!--      </ul>-->

      <fieldset>
        <legend>Add New Point</legend>

        <form @submit.prevent="createNewPoint">
          <label>
            Name
            <input type="text" required v-model="pointName" />
          </label>

          <button>Add at current time</button>
        </form>
      </fieldset>
      <fieldset>
        <legend>Add Segment</legend>
        <form @submit.prevent="addSegment">
          <label>
            Name
            <input type="text" required v-model="segmentName" />
          </label>

          <button v-if="Object.keys(segments).length !== 0" disabled>Add Segment</button>
          <button v-if="Object.keys(segments).length === 0" >Add Segment</button>
          <button v-if="Object.keys(segments).length === 0" disabled >Save Segment</button>
          <button v-if="Object.keys(segments).length !== 0" @click="saveSegments">Save Segment</button>
        </form>
      </fieldset>
    </main>
  </div>
</template>

<style>
body {
  font-family: sans-serif;
  font-size: 22px;
  line-height: 1.3;
}

h1 + p {
  margin-top: -1em;
  margin-bottom: 2em;
}

.playback {
  display: flex;
  justify-content: center;
  list-style: none;
  padding: 0;
  margin: 1.5em 0;
}
.playback button {
  margin: 0 .5em;
}
</style>

<script>
import Waveform from './components/Waveform.vue'
import { mapGetters } from 'vuex';

// Generated from https://colorbrewer2.org/#type=diverging&scheme=RdYlBu&n=6
const COLOR_PALETTE = ['#d73027','#fc8d59','#fee090','#e0f3f8','#91bfdb','#4575b4']

const slugify = (label) => String(label).toLocaleLowerCase().replace(/\s+/g, '-')

const randomColor = () => {
  return COLOR_PALETTE[ Math.floor(Math.random() * COLOR_PALETTE.length) ]
}

export default {
  components: {
    Waveform
  },

  data () {
    return {
      pointName: '',
      segmentName: '',
      segments: {},
      dataAudio: 'https://file-examples-com.github.io/uploads/2017/11/file_example_MP3_1MG.mp3'
    }
  },

  computed: {
    ...mapGetters('programme', ['currentTime', 'introTrack', 'extroTrack']),
  },

  methods: {
    // This trick enables us to trigger the `play()` method within the `Waveform.vue` component
    play (track) {
      this.$refs.waveform.play(track)
    },
    saveSegments () {
      this.segments['endTime'] = this.currentTime
      console.log('seg:', [this.segments])
      this.$store.commit('programme/addSegment', [this.segments])
      this.segments = {}

    },
    createNewPoint () {
      this.$store.commit('programme/addPoint', {
        labelText: this.pointName,
        id: slugify(this.pointName),
        color: randomColor(),
        time: this.currentTime,
        editable: true,
        removable: true
      })
      console.log(this.currentTime)

      this.pointName = ''
    },
    addSegment () {
      // this.$store.commit('programme/addPoint', {
      //   labelText: this.segmentName,
      //   // labelText: 'Test Segment',
      //   startTime: this.currentTime,
      //   endTime: this.currentTime + 10,
      //   editable: true,
      //   removable: true
      // })
       this.segments = {
        labelText: this.segmentName,
        // labelText: 'Test Segment',
        startTime: this.currentTime,
        // endTime: this.currentTime + 10,
        editable: true,
        removable: true
      }
      // console.log('seg:', this.segments)
      // this.segmentName = ''
    },
  },

  // created () {
  //   fetch('./points.json')
  //     .then(response => response.json())
  //     .then(points => this.$store.commit('programme/addPoints', points))
  // }
}

</script>
