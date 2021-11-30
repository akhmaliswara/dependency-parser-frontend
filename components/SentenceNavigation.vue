<template>
  <div v-if="visible" class="sentence-navigation">
    <a href="#" @click="$emit('previous')">❮</a>
      <input 
        class="sentence-navigation__input"
        type="number"
        :value="number"
        @keyup.enter="$emit('enter', $event)"
      />
      <span v-if="max !== 0">
        / {{max}}
      </span>
    <a href="#" @click="$emit('next')">❯</a>
  </div>
</template>

<script>
export default {
  props: {
    number: {
      type: Number,
      default: 1
    },
    visible: {
      type: Boolean,
      default: true
    },
    max: {
      type: Number,
      default: 0
    }
  }
}
</script>

<style scoped lang="scss">
.sentence-navigation {
  display: flex;
  align-items: center;
  justify-content: center;
  a {
    color: black;
    padding: 8px 16px;
    text-decoration: none;
    transition: background-color .3s;
    border: 1px solid #ddd;
  }
  &__input {
    -moz-appearance: textfield;
    width: 3rem;
  }
  &__input::-webkit-outer-spin-button,
  &__input::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }
}
</style>