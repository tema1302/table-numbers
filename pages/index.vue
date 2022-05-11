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
        // 'Mason Mount': 836694,
        Jorginho: 132874,
        'Mateo Kovačić': 136710,
        "N'Golo Kanté": 234148,
        'Ruben Loftus-Cheek': 284441,
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
  // background-image: url('./static/123.jpg');
  // background-repeat: no-repeat;
  // background-size: 130%;
  // background-attachment: fixed;
  // background-position: 47% 100%;
  background-image: linear-gradient(
    to right top,
    #d453d4,
    #bb62dc,
    #a16de0,
    #8876df,
    #707cdb,
    #5487e0,
    #3590e0,
    #0098de,
    #00a8e0,
    #00b7db,
    #00c3d0,
    #1fcfc3
  );
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
