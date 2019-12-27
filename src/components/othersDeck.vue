<template>
  <ul>
    <li
      v-for="(card, i) in deck.deckLength"
      :key="i"
    >
      
    </li>
  </ul>
</template>

<script>
export default {
  name: 'OthersDeck',
  props: [
    'deck'
  ],
}
</script>