<template>
  <div class="d-flex justify-content-center">
    <div v-if="!isRuning">
      <div class="d-flex flex-column mb-4 justify-content-center">
        <div class="mt-3">
          <label class="form-label">Delay before playing tone (in seconds)</label>
          <input class="form-control" type="text" v-model="delayBeforePlayingTone" />
        </div>
        <div class="mt-3">
          <label class="form-label">Delay after palying tone (in seconds)</label>
          <input class="form-control" type="text" v-model="delayAfterPlayingTone" />
        </div>
      </div>
      <div class="d-flex flex-row justify-content-center">
        <button class="btn btn-primary" @click="addOption">Add option</button>
        <button class="btn btn-success ms-2" @click="startRunning">Start</button>
      </div>

      <div
        class="mt-5 justify-content-center d-flex flex-column"
        style="overflow: hidden; height: 400px"
      >
        <div style="">
          <div v-for="note in notes" v-bind:key="note">
            <div class="d-flex flex-row mb-4 justify-content-center">
              <div>
                <label class="form-label">Pitch</label>
                <input class="form-control" type="text" v-model="note.pitch" />
              </div>
              <div>
                <label class="form-label">Rhythm</label>
                <select class="form-select" v-model="note.rhythm">
                  <option value="/h">1/2</option>
                  <option value="/q">1/4</option>
                  <option value="/8">1/8</option>
                </select>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-else>
      <button class="btn btn-danger" @click="stopRunning">Stop</button>
      <div id="music-note"></div>
    </div>
  </div>
</template>

<script>
import Vex from "vexflow";
import * as Tone from "tone";

export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  data() {
    return {
      delayBeforePlayingTone: 1,
      delayAfterPlayingTone: 1,
      notes: [],
      isRuning: false,
      playToneTimeout: null,
      triggerNextTimeout: null,
      vexFlowInstance: null,
    };
  },
  methods: {
    startRunning() {
      this.isRuning = true;

      this.$nextTick(() => {
        this.vexFlowInstance = new Vex.Flow.Factory({
          renderer: { elementId: "music-note", width: 500, height: 200 },
        });

        this.gameLoop(0);
      });
    },
    stopRunning() {
      clearTimeout(this.playToneTimeout);
      clearTimeout(this.triggerNextTimeout);
      this.isRuning = false;
    },
    gameLoop(prevIndex) {
      let index = prevIndex;

      while (prevIndex == index) {
        index = parseInt(Math.random() * this.notes.length);
      }

      let note = this.notes[index];

      this.renderMusic(note);

      this.playToneTimeout = setTimeout(() => {
        this.playNote(note);

        this.triggerNextTimeout = setTimeout(() => {
          this.gameLoop(index);
        }, this.delayAfterPlayingTone * 1000);
      }, this.delayBeforePlayingTone * 1000);
    },
    addOption() {
      this.notes.push({ value: "" });
    },
    renderMusic(note) {
      this.vexFlowInstance.getContext().clear();

      const score = this.vexFlowInstance.EasyScore();
      const system = this.vexFlowInstance.System();

      if (note.rhythm === "/8") {
        score.set({ time: "1/8" });
      } else if (note.rhythm === "/q") {
        score.set({ time: "1/4" });
      } else if (note.rhythm === "/h") {
        score.set({ time: "1/2" });
      }

      system
        .addStave({
          voices: [score.voice(score.notes(note.pitch + note.rhythm))],
        })
        .addClef("treble");
      this.vexFlowInstance.draw();
    },
    playNote(note) {
      const synth = new Tone.Synth().toDestination();
      synth.triggerAttackRelease(note.pitch, "8n");
    },
  },
  mounted() {
    this.notes = [
      { pitch: "A4", rhythm: "/q" },
      { pitch: "C4", rhythm: "/q" },
      { pitch: "B4", rhythm: "/q" },
      { pitch: "E4", rhythm: "/q" },
    ];
    /*var note = {pitch:'C4', rhythm:'/q'};
    this.renderMusic(note);
    this.playNote(note);*/
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
