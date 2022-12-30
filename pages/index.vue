<template lang="pug">
.container-fluid.px-10.mx-auto
  .min-h-screen.flex.items-start.justify-center
    PlayerFirst(v-for="(id, name, idx) in teams" :teamId="id" :playerName="name" :key="name")
</template>

<script>
import PlayerFirst from '~/components/PlayerFirst.vue'
export default {
  name: 'IndexPage',
  components: { PlayerFirst },
  data() {
    return {
      teams: {
        // 'Antonio Rüdiger': 142622,
        // Jorginho: 132874,
        // 'Mateo Kovačić': 136710,
        // "N'Golo Kanté": 234148,
        // 'Ruben Loftus-Cheek': 284441,
        // 'Marc Cucurella': 794939,
        // 'Mason Mount': 836694,
        // 'Kai Havertz': 836705,
        // 'Christian Pulišić': 817957,
        Chelsea: 38,
      },
    }
  },
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Montserrat&family=Tenor+Sans&family=Open+Sans+Condensed:wght@300&display=swap');

.container-fluid {
  // background-image: url('./static/123.jpg');
  background-repeat: no-repeat;
  background-size: cover;
  background-attachment: fixed;
  background-position: 47% 100%;
  // background-size: auto;
  background-image: url(https://kartinkin.net/uploads/posts/2021-07/1625811208_4-kartinkin-com-p-chernie-kubiki-oboi-krasivie-4.jpg);
  font-family: 'Tenor Sans', sans-serif;

  &::before {
    content: '';
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    z-index: 0;
    background: rgba($color: #222, $alpha: 0.1);
    // backdrop-filter: blur(1px);
    position: fixed;
  }
}
</style>
