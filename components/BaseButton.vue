<template>
  <v-btn @click="$emit('click', $event)">
    {{label}}
  </v-btn>
</template>

<script>
export default ({
  props: {
    label: {
      type: String,
      default: ''
    }
  }
})
</script>

<style scoped lang="scss">
.button {
  margin: 10rem;
  border: 1px solid black;
  background-color: red;
}
</style>