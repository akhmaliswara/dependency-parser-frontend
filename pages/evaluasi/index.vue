<template>
  <section class="evaluation">
    <h1>Evaluasi Hasil Parsing</h1>
    <h4>Silahkan masukkan file hasil parsing di bawah</h4>
    <v-form>
      Reference file:
      <input
          @input="goldFile = $event.target.files[0]"
          ref="input"
          type="file"
        >
      File yang akan dievaluasi:
      <input
          @input="inputFile = $event.target.files[0]"
          ref="input"
          type="file"
        >
      <p>Label yang dievaluasi (optional):</p>
      <v-autocomplete
        class="evaluation__input-autocomplete"
        dense
        :items="items"
        v-model="filterLabel"
      />
    </v-form>
    <BaseButton label="Submit" @click="onSubmit"/>
    <div v-if="loading" style="margin-top: 3rem">
      <v-progress-circular indeterminate />
    </div>
    <div v-if="showResult" style="margin-top: 3rem">
      <h4>Ikhtisar Evaluasi:</h4>
      <ul>
        <li>UAS: {{result.UAS.toFixed(2)}}</li>
        <li>LAS: {{result.LAS.toFixed(2)}}</li>
        <li>LA: {{result.LA.toFixed(2)}}</li>
      </ul>
      <br />
      <h4>Top Miss-Predicted Label (Actual Label: {{textLabel}})</h4>
      <ol>
        <li v-for="element in result.missLabel" :key="element[0]">
          {{element[0]}} (jumlah: {{element[1]}}, persentase: {{element[2].toFixed(3)}}%)
        </li>
      </ol>
      <br />
      <h4>List ID - Miss Predicted (ID Sentence, ID Word, Word)</h4>
      <ol>
        <li v-for="element in result.idMissList" :key="(element[0], element[1])">
          {{element[0]}} - {{element[1]}} - {{element[2]}}
        </li>
      </ol>
    </div>
  </section>
</template>

<script>
// import {displaCy} from 'displacy';

export default {
  layout: 'no-layout',
  data: () => ({
    goldFile: null,
    inputFile: null,
    loading: false,
    showResult: false,
    result: {},
    filterLabel: '',
    textLabel: '',
    items: [
      {text: '-Tanpa Filter-', value: ''}, 
      'nsubj:pass', 'nsubj', 'obj', 'iobj', 'obl', 'nmod', 'appos', 'nummod', 'csubj', 'csubj:pass', 'ccomp', 'xcomp', 'advcl', 'acl', 'advmod', 'amod', 'aux', 'cop', 'mark', 'det', 'case', 'conj', 'cc', 'fixed', 'flat', 'compound', 'compound:plur', 'parataxis', 'punct', 'root', 'dep'
    ]
  }),
  methods: {
    async onSubmit() {
      this.loading = true;
      this.showResult = false;
      const data = new FormData();
      data.append('gold_file', this.goldFile);
      data.append('input_file', this.inputFile);
      data.append('filter_label', this.filterLabel);

      const response = await this.$axios.post('/api/evaluate', data, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      });
      this.result = response.data;
      this.textLabel = this.filterLabel === '' ? 'Tanpa Filter' : this.filterLabel;
      this.showResult = true;
      this.loading = false;
      // this.visible = true;
      // this.visualize();
    },
    // visualize() {
    //   const displacy = new displaCy('http://localhost:8000', {
    //     container: '#displacy',
    //     format: 'spacy',
    //     distance: 180,
    //     offsetX: 50
    //   });

    //   displacy.render(this.parsedText[this.sentenceNumber - 1], {
    //     color: '#000000'
    //   });
    // },
    // onPrevious() {
    //   if (this.sentenceNumber !== 1) {
    //     this.sentenceNumber -= 1;
    //     this.visualize();
    //   }
    // },
    // onNext() {
    //   if (this.sentenceNumber !== this.parsedText.length) {
    //     this.sentenceNumber += 1;
    //     this.visualize();
    //   }
    // },
    // onEnter($event) {
    //   const value = $event.target.valueAsNumber;
    //   if ((value >= 1) && (value <= this.parsedText.length)) {
    //     this.sentenceNumber = value;
    //     this.visualize();
    //   }
    // }
  }
}
</script>

<style scoped lang="scss">
.evaluation {
  &__input-autocomplete {
    width: 8rem;
  }
}
.visualizer {
  border: 1px solid black;
}

#displacy {
  overflow-x: scroll;
}
</style>