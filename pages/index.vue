<template lang="pug">
.container-fluid.px-10.mx-auto
  .min-h-screen.flex.items-start.justify-center
    PlayerFirst(v-for="(id, name, idx) in players" :playerId="id" :playerName="name" :key="name")
</template>

<script>
import PlayerFirst from '~/components/PlayerFirst.vue'
export default {
  name: 'IndexPage',
  components: { PlayerFirst },
  data() {
    return {
      players: {
        'Odilon Kossounou': 973437,
        'Edmond Tapsoba': 883262,
        'Jules Koundé': 827212,
        // Neymar: 124712,
        // 'Kylian Mbappé': 826643,
        // 'Jérémie Boga': 367056,
        // 'Mohamed Salah': 159665,
        // 'Yannick Carrasco': 182001,
        // 'Jack Grealish': 189061,
        // 'Raheem Sterling': 138534,
      },
    }
  },
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Montserrat&family=Tenor+Sans&family=Open+Sans+Condensed:wght@300&display=swap');

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
