<template>
  <div class="sequencer">
    <h2>The Sequencer</h2>
    <div class="sequencer-control">
      <button class="base-btn" @click="toggleStart">{{ playButtonText }}</button>
      <div class="bpm">
        <h3 style="margin-bottom: 0;">BPM: {{ bpm }}</h3>
        <input v-model="bpm" @mouseup="updateBPM" type="range" name="bpm" id="bpm" min="60" max="200">
      </div>
      <button @click="clearAllSequences" class="base-btn clear-btn">Clear</button>
    </div>
    <div
      class="instrument-row"
      v-for="instrument in instrumentCount"
      :key="instrument"
    >
      <Trigger
        class="step-btn"
        v-for="step in steps"
        :key="step"
        :triggerInfo="{instrument: instrument, step: step}"
        :clearAllTriggers="shouldClearSequences"
        @trigger-state="updateSequence"
      ></Trigger>
    </div>
    <div class="instrument-row">
      <StepVisualizer
        class="step-visualizer"
        v-for="step in steps"
        :key="step"
        :stepNumber="step"
        :isCurrentStep="step === currentStep"
      ></StepVisualizer>
    </div>
  </div>
</template>

<script>
import * as Tone from 'tone'
import Trigger from './Trigger.vue'
import StepVisualizer from './StepVisualizer.vue'

export default {
  name: 'Sequencer',
  components: {
    Trigger,
    StepVisualizer
  },
  props: {

  },
  data() {
    return {
      audioContextStarted: false,
      isPlaying: false,
      playButtonText: 'Play',
      instrumentCount: 4,
      instrumentPitches: ['A5', 'A4', 'G3', 'D3'],
      steps: 16,
      currentStep: 0,
      stepSubdivision: 8,
      shouldClearSequences: false,
      bpm: null
    }
  },
  computed: {
    getBPM() {
      return Tone.Transport.bpm.value
    }
  },
  methods: {
    toggleStart() {
      if (!this.audioContextStarted) {
        Tone.start().then(() => {
          this.audioContextStarted = true
          console.log('audio context started!')
          this.setupStepVisualizer()
        })
      }
      this.isPlaying = !this.isPlaying
      this.playButtonText = this.isPlaying ? 'Stop' : 'Play'
      if (this.isPlaying) {
        this.currentStep = 0
        Tone.Transport.start()

      } else if (!this.isPlaying) {
        Tone.Transport.stop()
        this.currentStep = -1
      }
    },
    updateSequence(triggerIsOn, info) {
      // If all sequences have previously been cleared, set the boolean value that toggles this action to true on first update
      if (this.shouldClearSequences) {
        this.shouldClearSequences = false
      }

      // Get the instrument number from the trigger that was clicked
      const instrument = info.instrument - 1
      // Set the update value to a note if trigger on, and a rest (null) if not
      const updateVal = triggerIsOn ? this.instrumentPitches[instrument] : null
      // Update the relevent instrument sequence event in it's events array
      this.instrumentSequences[instrument].events[info.step - 1] = updateVal
      // console.log(this.instrumentSequences[instrument].events)
    },
    clearAllSequences() {
      // Set event sequence array for all instruments to null
      this.instrumentSequences.forEach(instrument => {
        instrument.events = Array(this.steps).fill(null)
      })
      // Set shouldClearSequences to true to send update to Trigger components and clear "trigger-on" styling
      this.shouldClearSequences = true
    },
    setupStepVisualizer() {
      // Schedule a repeated call to Tone.Draw on the set beat (stepSubdivision)to update the current step - this will update the class name on StepVisualizer and visualize the current step position
      Tone.Transport.scheduleRepeat((time) => {
        Tone.Draw.schedule(() => {
          this.currentStep = this.currentStep % this.steps + 1
        }, time)
      }, this.stepSubdivision + 'n')
    },
    updateBPM() {
      Tone.Transport.bpm.value = this.bpm
    }
  },
  created() {
    // When Sequencer component is created create a synth and the Tone sequences for each instrument. Set up some base Tone.Transport properties

    // Create a symth for each instrument and set some inital properties
    this.synths = Array(this.instrumentCount).fill(null).map(() => {
      const synth = new Tone.Synth()
      // High volume was causing clicking/CPU issue, lower it!
      synth.volume.value = -10
      return synth
    })
    
    // Set the "instrumentSequences" as an array of null values of length "instrumentCount" ("instrumentCount" = the number of rows in our sequencer)
    this.instrumentSequences = Array(this.instrumentCount).fill(null)

    // Map over the existing "instrumentSequences" array to create a Tone Sequence for each instrument with "steps" number of events ("steps" = the number of columns in our sequencer)
    // https://tonejs.github.io/docs/14.7.77/Sequence
    this.instrumentSequences = this.instrumentSequences.map((d, i) => {
      return new Tone.Sequence((time, note) => {
        this.synths[i].toDestination().triggerAttackRelease(
          note, this.stepSubdivision * 8 + 'n', time
        )
      }, Array(this.steps).fill(null), this.stepSubdivision + 'n').start(0)
    })

    // Set the initial bpm of the Transport
    this.bpm = Tone.Transport.bpm.value
  },
  beforeUnmount() {
    // Cancel any scheduled events in Transport
    Tone.Transport.cancel()
    // Dispose of the instruments
    this.instrumentSequences.forEach(instrument => instrument.dispose())
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .base-btn {
    width: 6.5rem;
    height: 3rem;
    padding: 0.5rem;
    border: none;
    background-color: lightcoral;
    cursor: pointer;
    font-size: 1rem;
    margin: 0 0.25rem;
  }

  .bpm {
    text-align: center;
  }

  .clear-btn {
    margin-left: auto;
  }

  .sequencer-control {
    display: flex;
    flex-direction: row;
    justify-content: left;
    align-items: center;
  }

  .instrument-row {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .step-visualizer:nth-child(4n+1) {
    font-weight: bold;
    font-size: large;
  }
</style>
