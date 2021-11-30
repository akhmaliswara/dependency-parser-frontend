<template>
  <section>
    <h1>Perbandingan Hasil Parsing</h1>
    <h4>Silahkan masukkan file hasil parsing di bawah</h4>
    <v-form>
      File yang akan dibandingkan:
      <input
          @input="inputFile = $event.target.files[0]"
          ref="input"
          type="file"
        >
      File pembanding:
      <input
          @input="comparedFile = $event.target.files[0]"
          ref="input"
          type="file"
        >
      <br/>
      Reference file:
      <input
          @input="goldFile = $event.target.files[0]"
          ref="input"
          type="file"
        >
      <p>Label yang dievaluasi (optional):</p>
      <v-autocomplete
        class="comparison__input-autocomplete"
        dense
        :items="items"
        v-model="filterLabel"
      />
    </v-form>
    <BaseButton label="Submit" @click="onSubmit"/>
    <div v-if="showResult" style="margin-top: 3rem">
      <h4>Ikhtisar</h4>
      <ul>
        <li>total kalimat yg unggul (yg pembanding salah): {{result.totalID}}</li>
        <li>total kata yg unggul (yg pembanding salah): {{result.totalWord}}</li>
      </ul>
      <br/>
      <h4>List kata yg unggul (ID - List[wordID, kata])</h4>
      <ul> 
        <li v-for="items in result.list" :key="items[0]">
          ID: {{items[0]}} - {{items[1]}}
        </li>
      </ul>
    </div>
    <BaseButton v-if="showResult" label="Visualisasikan" @click="visualize"/>
    <div v-if="showResult">
      <SentenceNavigation
        :visible="visible"
        :number="page"
        :max="result.list.length"
        @enter="onEnter"
        @previous="onPrevious"
        @next="onNext"
      />
      <div>
        <h2 v-if="visible">ID = {{result.list[page - 1][0]}}</h2>
        <h4 v-if="visible">Visualisasi Input</h4>
        <div id="displacyInput" class="visualizer"></div>
        <h4 v-if="visible">Visualisasi Pembanding</h4>
        <div id="displacyCompared" class="visualizer"></div>
        <h4 v-if="visible">Visualisasi Reference</h4>
        <div id="displacyGold" class="visualizer"></div>
      </div>
    </div>
  </section>
</template>

<script>
import {displaCy} from 'displacy';

export default {
  layout: 'no-layout',
  data: () => ({
    goldFile: null,
    inputFile: null,
    comparedFile: null,
    loading: false,
    showResult: false,
    result: {},
    filterLabel: '',
    textLabel: '',
    items: [
      {text: '-Tanpa Filter-', value: ''}, 
      'nsubj:pass', 'nsubj', 'obj', 'iobj', 'obl', 'nmod', 'appos', 'nummod', 'csubj', 'csubj:pass', 'ccomp', 'xcomp', 'advcl', 'acl', 'advmod', 'amod', 'aux', 'cop', 'mark', 'det', 'case', 'conj', 'cc', 'fixed', 'flat', 'compound', 'compound:plur', 'parataxis', 'punct', 'root', 'dep'
    ],
    visible: false,
    page: 1
  }),
  methods: {
    async onSubmit() {
      this.loading = true;
      this.showResult = false;
      const data = new FormData();
      data.append('gold_file', this.goldFile);
      data.append('input_file', this.inputFile);
      data.append('compared_file', this.comparedFile);
      data.append('filter_label', this.filterLabel);

      const response = await this.$axios.post('/api/compare', data, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      });
      this.result = response.data;
      this.showResult = true;
      this.loading = false;
    },
    async visualize() {
      console.log(this.result.list.length);
      if (this.result.list.length !== 0) {
        this.visible = true;

        const displacyInput = this.initDisplacy();
        displacyInput.container = this.$el.querySelector('#displacyInput');
        const displacyCompared = this.initDisplacy();
        displacyCompared.container = this.$el.querySelector('#displacyCompared');
        const displacyGold = this.initDisplacy();
        displacyGold.container = this.$el.querySelector('#displacyGold');
        
        const parsedInput = await this.parsedText('input');
        const parsedCompared = await this.parsedText('compared');
        const parsedGold = await this.parsedText('gold');

        this.renderDisplacy(displacyInput, parsedInput);
        this.renderDisplacy(displacyCompared, parsedCompared);
        this.renderDisplacy(displacyGold, parsedGold);
      }
    },
    async parsedText(type) {
      const data = new FormData();
      if (type === 'input') data.append('input_file', this.inputFile);
      else if (type === 'compared') data.append('input_file', this.comparedFile);
      else data.append('input_file', this.goldFile);
      data.append('id', this.result.list[this.page - 1][0]);
      console.log(this.result.list[this.page - 1][0]);
      const response = await this.$axios.post('/api/visualize', data, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      });

      return response.data;
    },
    initDisplacy() {
      return new displaCy('http://localhost:8000', {
        format: 'spacy',
        distance: 200,
        offsetX: 50
      });
    },
    renderDisplacy(container, parsedText) {
      console.log(container);
      console.log(parsedText);
      container.render(parsedText, {
        color: '#000000'
      });
    },
    onPrevious() {
      if (this.page !== 1) {
        this.page -= 1;
        this.visualize();
      }
    },
    onNext() {
      if (this.page !== this.result.list.length) {
        this.page += 1;
        this.visualize();
      }
    },
    onEnter($event) {
      const value = $event.target.valueAsNumber;
      if ((value >= 1) && (value <= this.result.list.length)) {
        this.page = value;
        this.visualize();
      }
    }
  }
}
</script>

<style scoped lang="scss">
.comparison {
  &__input-autocomplete {
    width: 8rem;
  }
}
.visualizer {
  border: 1px solid black;
  overflow-x: scroll;
}
</style>