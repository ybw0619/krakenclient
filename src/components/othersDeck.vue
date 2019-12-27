<template>
  <div>
    <p>userId: {{deck.user}}</p>
    <ul>
      <li
        v-for="(card, i) in deck.deckLength"
        :key="i"
        @click="selectCard(i)"
      >
        [뒷면]
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'OthersDeck',
  props: [
    'deck'
  ],
  methods: {
    selectCard(i) {
      this.$emit('select-card', {user: this.deck.user, ci: i})
    },
  },
}
</script>