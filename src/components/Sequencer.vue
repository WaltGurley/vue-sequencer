<template>
  <h2>The Sequencer</h2>
  <button class="play-btn" @click="toggleStart">{{ playButtonText }}</button>
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
      @trigger-state="updateSequence"
    ></Trigger>
  </div>
  <div class="instrument-row">
    <StepVisualizer
      v-for="step in steps"
      :key="step"
      :isCurrentStep="step === currentStep"
    ></StepVisualizer>
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
      stepSubdivision: '8n',
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
        Tone.Transport.start('+0.1')

      } else if (!this.isPlaying) {
        Tone.Transport.stop()
        this.currentStep = -1
      }
    },
    updateSequence(triggerIsOn, info) {
      // Get the instrument number from the trigger that was clicked
      const instrument = info.instrument - 1
      // Set the update value to a note if trigger on, and a rest (null) if not
      const updateVal = triggerIsOn ? this.instrumentPitches[instrument] : null
      // Update the relevent instrument sequence event in it's events array
      this.instrumentSequences[instrument].events[info.step - 1] = updateVal
      // console.log(this.instrumentSequences[instrument].events)
    },
    setupStepVisualizer() {
      Tone.Transport.scheduleRepeat((time) => {
        Tone.Draw.schedule(() => {
          this.currentStep = this.currentStep % this.steps + 1
        }, time)
      }, this.stepSubdivision)
      console.log(this.synths)
    }
  },
  created() {
    // When component is created create a synth and the Tone sequences for each instrument

    // Create a symth for each instrument and set some inital properties
    this.synths = new Array(this.instrumentCount).fill(null).map(() => {
      const synth = new Tone.Synth()
      // High volume was causing clicking/CPU issue, lower it!
      synth.volume.value = -10
      return synth
    })
    
    // Set the "instrumentSequences" as an array of null values of length "instrumentCount" ("instrumentCount" = the number of rows in our sequencer)
    this.instrumentSequences = new Array(this.instrumentCount).fill(null)

    // Map over the existing "instrumentSequences" array to create a Tone Sequence for each instrument with "steps" number of events ("steps" = the number of columns in our sequencer)
    // https://tonejs.github.io/docs/14.7.77/Sequence
    this.instrumentSequences = this.instrumentSequences.map((d, i) => {
      return new Tone.Sequence((time, note) => {
        this.synths[i].toDestination().triggerAttackRelease(note, '64n', time)
      }, new Array(this.steps).fill(null), this.stepSubdivision).start(0)
    })
    console.log(this.instrumentSequences, this.instrumentCount)
  },
  unmounted() {
    Tone.Transport.cancel()
    this.instrumentSequences.forEach(instrument => instrument.dispose())
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .play-btn {
    width: 6rem;
    padding: 0.5rem;
    font-size: 1rem;
  }

  .instrument-row {
    display: flex;
    align-items: center;
    justify-content: center;
  }
</style>
