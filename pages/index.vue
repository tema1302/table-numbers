<template lang="pug">
  .container-fluid.p-10.mx-auto
    .flex.items-center.justify-center
      .stats-table
        .seasons
          .seasons-item 18/19
          .seasons-item 19/20
          .seasons-item 20/21
          .seasons-item 21/22
        //- .xG
        //- .xA
        .matches(v-for="(matchesInSeason, idx) in matches" :key="idx")
          .matches-item {{ matchesInSeason }}
        .injures(v-for="(injuresInSeason, idx) in injures" :key="idx")
          .injures-item {{ injuresInSeason }}
</template>

<script>
export default {
  name: 'IndexPage',
  data () {
    return {
      parser: new DOMParser(),
      matches: [],
      injures: [],
      years: [2018, 2019, 2020, 2021]
    }
  },
  computed: {},
  async beforeMount() {
    await this.fetchMatches()
  },
  methods: {
    async fetchMatches() {
      try {
        // this.years.forEach(async year => {
        //   const request = await this.$axios.get(`/niklas-sule/leistungsdaten/spieler/166601/plus/0?saison=${year}`)
        //   const htmlString = request.data
        //   console.log(htmlString)
        //   const doc = this.parser.parseFromString(htmlString, 'application/xml')
        //   console.log(doc)
        //   const number = doc.querySelector('#yw1 > table > tfoot > tr > td:nth-child(3)')
        //   console.log(number)
        //   this.matches.push(number)
        // })

        const request = await this.$axios.get('/player/romelu-lukaku/78893')
        const htmlString = request.data
        console.log(htmlString)
        const doc = this.parser.parseFromString(htmlString, 'application/xml')
        console.log(doc)
        const number = doc.querySelector('#__next > main > div > div.Content__PageContainer-sc-g01dez-0.eyhylD > div > div.Col-sc-ffl2sp-0.styles__SidebarCol-sc-1loq6tv-3.fWoEem > div.styles__SeasonSection-sc-5fu9xh-0.kMUwUo > div.styles__StatisticsPanel-sc-5fu9xh-1.fsezkc > div > div:nth-child(4) > table > tbody > tr:nth-child(7) > td:nth-child(2)')
        console.log(number)
        this.matches.push(number)
      } catch (e) {
        console.error(e)
      }

    }
  }
}
</script>
