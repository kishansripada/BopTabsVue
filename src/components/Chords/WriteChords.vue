<template>
  <div>
    <button
      @click="autofill()"
      style="
        position: absolute;
        right: 110px;
        margin-top: -40px;
        border: solid gray 1px;
      "
      type="button"
      :class="{
        btn: true,
      }"
    >
      Autofill
    </button>
    <button
      @click="clear()"
      style="
        position: absolute;
        right: 220px;
        margin-top: -40px;
        border: solid gray 1px;
      "
      type="button"
      :class="{
        btn: true,
      }"
    >
      Clear
    </button>

    <button
      @click="trimStartBeat"
      style="
        position: absolute;
        left: 20px;
        margin-top: 30px;
        border: solid gray 1px;
      "
      type="button"
      :class="{
        btn: true,
      }"
    >
      Trim 1
    </button>

    <div class="container mt-4">
      <div class="row row-cols-1 row-cols-md-8 g-4">
        <div class="col" v-for="(beat, index) in userInputChords" :key="index">
          <div>
            <div class="card" v-on:click="changePosition(beat)">
              <button type="button" :class="{ btn: true, 'btn-warning': true }">
                {{ "  " }}
              </button>
            </div>

            <select
              aria-label=".form-select-sm example"
              v-model="userInputChords[index].chord"
              class="card"
              :style="[
                index == currentBar
                  ? { border: 'solid gray 3px' }
                  : { background: '#FFF' },
              ]"
              :class="{
                'form-control': true,
                'form-select-sm': true,
              }"
            >
              <option selected></option>
              <option
                :value="chord"
                v-for="(chord, index) in chords"
                :key="index"
              >
                {{ chord }}
              </option>
            </select>
          </div>
        </div>
      </div>

      <div class="d-flex flex-fill pt-5">
        <button class="btn btn-primary mt-auto me-auto" v-on:click="cancel()">
          Cancel
        </button>
        <button
          class="btn btn-primary ms-auto mt-auto"
          v-on:click="postChords()"
        >
          Submit
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
import { mapFields } from "vuex-map-fields";
import { App, Credentials } from "realm-web";
export default {
  name: "WriteChords",
  computed: {
    ...mapState(["currentTrack", "spotifyPosition", "isWritingChords"]),
    currentBar() {
      return this.userInputChords.findIndex((beat) => {
        return this.spotifyPosition / 1000 < beat.start;
      });
    },
  },
  data() {
    return {
      user: null,
      chords: [
        "C",
        "C#",
        "D",
        "Eb",
        "E",
        "F",
        "F#",
        "G",
        "Ab",
        "A",
        "Bb",
        "B",
        "Cm",
        "C#m",
        "Dm",
        "Ebm",
        "Em",
        "Fm",
        "F#m",
        "Gm",
        "Abm",
        "Am",
        "Bbm",
        "Bm",
      ],
      userInputChords: [],
      description: "",
    };
  },
  async created() {
    this.userInputChords = this.currentTrack.trackAnalysis.beats.slice();

    const app = new App({ id: "boptabs-wwrqq" });
    const credentials = Credentials.anonymous();
    const user = await app.logIn(credentials);
    this.user = user;
  },
  methods: {
    changePosition(beat) {
      this.$store.commit("setChordPosition", beat.start);
    },
    trimStartBeat() {
      this.userInputChords.shift();
    },
    cancel() {
      this.$store.commit("setIsWritingChords", false);
    },
    postChords() {
      let track = JSON.parse(JSON.stringify(this.currentTrack));
      console.log(track);

      let chordObject = {
        approved: false,
        chords: this.userInputChords,
        rating: null,
        author: JSON.parse(localStorage.token).user.id,
        description: this.description,
        dateSubmitted: new Date(),
      };

      if (track.chords && track.tabs) {
        // if track exists
        this.user.functions.addNewChords(track.id, chordObject).then(() => {
          location.reload();
        });
      } else {
        // if track doesn't exist
        let document = {
          name: track.name,
          primaryArtist: track.artists.map((artist) => artist.name).join(", "),
          spotifyId: track.id,
          chords: [chordObject],
          tabs: [],
        };

        this.user.functions.addOne(document).then(() => {
          location.reload();
        });
      }
    },
    autofill() {
      const makeRepeated = (arr, repeats) =>
        Array.from({ length: repeats }, () => arr).flat();

      let chords = this.userInputChords.slice();
      let reversed = chords.slice().reverse();

      if (chords.filter((beat) => beat.chord).length > 2) {
        let firstChordIndex = chords.findIndex((beat) => beat.chord);
        let lastChordIndex =
          chords.length - reversed.findIndex((beat) => beat.chord) - 1;

        let chunk = chords
          .map((beat) => beat.chord)
          .slice(firstChordIndex, lastChordIndex);

        let outputChords = [
          ...new Array(firstChordIndex).fill(undefined),
          ...makeRepeated(chunk, chords.length / chunk.length),
        ];
        this.userInputChords = this.userInputChords.map((chord, index) => {
          return {
            ...chord,
            chord: outputChords[index],
          };
        });
      }
    },
    clear() {
      this.userInputChords = this.currentTrack.trackAnalysis.beats.slice();
    },
  },
};
</script>

<style scoped>
.outline {
  border: "solid gray 5px";
}
.row-cols-md-8 > * {
  flex: 0 0 auto;
  width: 6.25%;
}
</style>