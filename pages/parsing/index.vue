<template>
  <section>
    <h1>Parsing Kalimat Berbahasa Indonesia</h1>
    <h4>Silahkan masukkan kalimat yang ingin diparse</h4>
    <v-form>
      <v-text-field 
        v-model="sentence"
      />
      Parser:
      <v-autocomplete
        class="parsing__input-autocomplete"
        dense
        multiple
        :items="items"
        v-model="filterParser"
      />
    </v-form>
    <BaseButton label="Submit" @click="onSubmit"/>
    <div id="visualizer"></div>
  </section>
</template>

<script>
import {displaCy} from 'displacy';

export default {
  layout: 'no-layout',
  data: () => ({
    sentence: '',
    filterParser: '',
    items: ['stanza', 'spacy', 'BIST Transition-based', 'MST Parser', 'Malt Parser']
  }),
  methods: {
    async onSubmit() {
      const data = {sentence: this.sentence, parser: this.filterParser};
      const response = await this.$axios.post('/api/parse', data);
      console.log(response);
      this.visualize(response.data[0])
    },
    visualize(parsedText) {
      const displacy = new displaCy('http://localhost:8000', {
        format: 'spacy',
        distance: 200,
        offsetX: 50,
        container: '#visualizer'
      });

      displacy.render(parsedText, {
        color: '#000000'
      });
    }
  }
}
</script>

<style lang="scss" scoped>
.parsing {
  &__input-autocomplete {
    width: 10rem;
  }
}
</style>