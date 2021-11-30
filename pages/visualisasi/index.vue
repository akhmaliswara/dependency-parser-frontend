<template>
  <section>
    <h1>Visualisasi Hasil Parsing</h1>
    <h4>Silahkan masukkan file hasil parsing di bawah</h4>
    <v-form>
      <input
          @input="inputFile = $event.target.files[0]"
          ref="input"
          type="file"
        >
    </v-form>
    <BaseButton label='Submit' @click="onSubmit"/>
    <div class="visualizer">
      <SentenceNavigation
        :visible="visible"
        :number="sentenceNumber"
        :max="parsedText.length"
        @enter="onEnter"
        @previous="onPrevious"
        @next="onNext"
       />
      <div v-if="loading" style="margin-top: 3rem">
        <v-progress-circular indeterminate />
      </div>
      <div id="displacy">
      </div>
    </div>
  </section>
</template>

<script>
import {displaCy} from 'displacy';

export default {
  layout: 'no-layout',
  data: () => ({
    inputFile: null,
    parsedText: {},
    loading: false,
    sentenceNumber: 1,
    visible: false,
  }),
  mounted() {
    // this.testGet();
  },
  methods: {
    async testGet() {
      const response = await this.$axios.get("/api/");
    },
    async onSubmit() {
      this.loading = true;

      const data = new FormData();
      const file = this.inputFile;
      data.append('input_file', file)
      const response = await this.$axios.post('/api/visualize', data, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      });
      this.parsedText = response.data;
      
      this.loading = false;
      this.visible = true;
      this.visualize();
    },
    visualize() {
      const displacy = new displaCy('http://localhost:8000', {
        container: '#displacy',
        format: 'spacy',
        distance: 180,
        offsetX: 50
      });

      displacy.render(this.parsedText[this.sentenceNumber - 1], {
        color: '#000000'
      });
    },
    onPrevious() {
      if (this.sentenceNumber !== 1) {
        this.sentenceNumber -= 1;
        this.visualize();
      }
    },
    onNext() {
      if (this.sentenceNumber !== this.parsedText.length) {
        this.sentenceNumber += 1;
        this.visualize();
      }
    },
    onEnter($event) {
      const value = $event.target.valueAsNumber;
      if ((value >= 1) && (value <= this.parsedText.length)) {
        this.sentenceNumber = value;
        this.visualize();
      }
    }
  }
}
</script>

<style scoped lang="scss">
.visualizer {
  border: 1px solid black;
}

#displacy {
  overflow-x: scroll;
}
</style>