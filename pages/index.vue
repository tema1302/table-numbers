<template lang="pug">
.container-fluid.px-10.mx-auto
  .min-h-screen.flex.items-start.justify-center
    PlayerFirst(v-for="(id, name, idx) in players" :playerId="id" :playerName="name" :key="idx")
</template>

<script>
import PlayerFirst from '~/components/PlayerFirst.vue'
export default {
  name: 'IndexPage',
  components: { PlayerFirst },
  data() {
    return {
      players: {
        'Н’Голо Канте': 234148,
        'Деклан Райс': 856714,
        'Орельен Тчуамени': 859025,
      },
    }
  },
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Montserrat&family=Tenor+Sans&family=Open+Sans+Condensed:wght@300&display=swap');
$green: #07d0186e;
$yellow: #faf20182;
$orange: #fa8e01c9;
$red: #fa260193;

.container-fluid {
  background-image: url('./static/123.jpg');
  background-repeat: no-repeat;
  background-size: 100%;
  background-position: 47% 100%;
  font-family: 'Tenor Sans', sans-serif;

  &::before {
    content: '';
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    z-index: 0;
    background: rgba($color: #222, $alpha: 0.5);
    // backdrop-filter: blur(1px);
    position: fixed;
  }
}
</style>
