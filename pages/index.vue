<template lang="pug">
  .container-fluid.px-10.mx-auto
    input(type="text", v-model="playerId", @change="fetchSeasons")
    .min-h-screen.flex.items-center.justify-center
      .stats-table.text-center.font-sans.text-sm.grid.grid-cols-6
        .season
          .season.mx-1.py-3.stat-legend Сезоны
          .season.mx-1.py-3(v-for="(season, idx) in seasonsItem", :key="idx") {{ season }}
        //- .xG
        //- .xA
        //- .teams
        //-   .team.mx-1.py-3.stat-legend Команда
        //-   .team.mx-1.py-3(v-for="(team, idx) in teams", :key="idx") {{ team }}
        .matches
          .match.mx-1.py-3.stat-legend Матчей в сезоне
          .match.mx-1.py-3(v-for="(appearance, idx) in overall.appearances" :key="idx") {{ appearance }}
        .matches
          .match.mx-1.py-3.stat-legend Из них в старте
          .match.mx-1.py-3(v-for="(matchesInSeason, idx) in overall.matchesStarted" :key="idx") {{ matchesInSeason }}
        .cols-comparison.touches
          .stat-legend.touch.mx-1.py-3 Касаний / 90мин
          .stat-value.touch.mx-1.py-3(v-for="(touches, idx) in overall.touches" :key="idx")
            span.range {{ touches }}
              span.range-width(:style="{ width: overallWidth.touches[idx] + '%' }")
        .cols-comparison.totalPasses
          .stat-legend.accurateP.mx-1.py-3 Пасов / 90мин
          .stat-value.accurateP.mx-1.py-3(v-for="(totalPasses, idx) in overall.totalPasses" :key="idx")
            span.range {{ totalPasses }}
              span.range-width(:style="{ width: overallWidth.totalPasses[idx] + '%' }")
        .cols-comparison.accuratePassesPercentage
          .stat-legend.accuratePP.mx-1.py-3 % успешных пасов / 90мин
          .stat-value.accuratePP.mx-1.py-3(v-for="(accuratePassesPercentage, idx) in overall.accuratePassesPercentage" :key="idx")
            span.range {{ accuratePassesPercentage }}%
              span.range-width(:style="{ width: overallWidth.accuratePassesPercentage[idx] + '%' }")
        .cols-comparison.aerialDuelsWonPercentage
          .stat-legend.aerialDWP.mx-1.py-3 % выигранных воздушных единоборств
          .stat-value.aerialDWP.mx-1.py-3(v-for="(aerialDuelsWonPercentage, idx) in overall.aerialDuelsWonPercentage" :key="idx")
            span.range {{ aerialDuelsWonPercentage }}%
              span.range-width(:style="{ width: overallWidth.aerialDuelsWonPercentage[idx] + '%' }")
        .cols-comparison.groundDuelsWon
          .stat-legend.groundDW.mx-1.py-3 Выигранных наземных дуэлей
          .stat-value.groundDW.mx-1.py-3(v-for="(groundDuelsWon, idx) in overall.groundDuelsWon" :key="idx")
            span.range {{ groundDuelsWon }}
              span.range-width(:style="{ width: overallWidth.groundDuelsWon[idx] + '%' }")
        .cols-comparison.groundDuelsWonPercentage
          .stat-legend.groundDW.mx-1.py-3 % успешных наземных дуэлей
          .stat-value.groundDW.mx-1.py-3(v-for="(groundDuelsWonPercentage, idx) in overall.groundDuelsWonPercentage" :key="idx")
            span.range {{ groundDuelsWonPercentage }}%
              span.range-width(:style="{ width: overallWidth.groundDuelsWonPercentage[idx] + '%' }")
        .cols-comparison.longBalls
          .stat-legend.lB.mx-1.py-3 Длинных передачи / 90мин
          .stat-value.lB.mx-1.py-3(v-for="(totalLongBalls, idx) in overall.totalLongBalls" :key="idx")
            span.range {{ totalLongBalls }}
              span.range-width(:style="{ width: overallWidth.totalLongBalls[idx] + '%' }")
        .cols-comparison.accurateLongBallsPercentage
          .stat-legend.lB.mx-1.py-3 % успешных длинных передач / 90мин
          .stat-value.lB.mx-1.py-3(v-for="(accurateLongBallsPercentage, idx) in overall.accurateLongBallsPercentage" :key="idx")
            span.range {{ accurateLongBallsPercentage }}%
              span.range-width(:style="{ width: overallWidth.accurateLongBallsPercentage[idx] + '%' }")
        .cols-comparison.interceptions
          .stat-legend.interc.mx-1.py-3 Перехватов / 90мин
          .stat-value.interc.mx-1.py-3(v-for="(interceptions, idx) in overall.interceptions" :key="idx")
            span.range {{ interceptions }}
              span.range-width(:style="{ width: overallWidth.interceptions[idx] + '%' }")
        .cols-comparison.tackles
          .stat-legend.accurateP.mx-1.py-3 Отборов за игру
          .stat-value.accurateP.mx-1.py-3(v-for="(tackles, idx) in overall.tackles" :key="idx")
            span.range {{ +tackles.toFixed(1) }}
              span.range-width(:style="{ width: overallWidth.tackles[idx] + '%' }")
        .cols-comparison.tacklesOn90
          .stat-legend.accurateP.mx-1.py-3 % успешных отборов за игру
          .stat-value.accurateP.mx-1.py-3(v-for="(tacklesOn90, idx) in overall.tacklesOn90" :key="idx")
            span.range {{ tacklesOn90 }}%
              span.range-width(:style="{ width: overallWidth.tacklesOn90[idx] + '%' }")
        .cols-comparison.possessionLost
          .stat-legend.possessionL.mx-1.py-3 Утрачено владение / 90мин
          .stat-value.possessionL.mx-1.py-3(v-for="(possessionLost, idx) in overall.possessionLost" :key="idx")
            span.range {{ possessionLost }}
              span.range-width(:style="{ width: overallWidth.possessionLost[idx] + '%' }")
</template>

<script>
export default {
  name: 'IndexPage',
  data () {
    return {
      playerId: '',
      seasonTournamentId: [],
      seasonTournamentYears: [],
      seasonsItem: ['2019/2020', '2020/2021', '2021/2022'],
      // seasonsItem: ['2018/2019', '2019/2020', '2020/2021', '2021/2022'],
      years: [2018, 2019, 2020, 2021],
      // sofascore. Take an average values in this tournament
      bundesIdSofa: 35,
      bundesSeasonsSofa: [17597, 23538, 28210, 37166],
      premLeague: 17,
      // premLeagueSeasonsSofa: [17359, 23776, 29415, 37036],
      premLeagueSeasonsSofa: [23776, 29415, 37036],
      chLeagueIdSofa: 7,
      // chLeagueSeasonsSofa: [17351, 23766, 29267, 36886], for Sule
      chLeagueSeasonsSofa: [23766, 29267, 36886],
      suleId: 168937,
      rudigerId: 142622,

      // sofa stats
      teams: [],
      leagues: [
        {
          17: [17359, 23776, 29415, 37036]
        },
        {
          7: [23766, 29267, 36886]
        }
      ],
      statNames: [
        'appearances',
        'matchesStarted',
        'touches',
        'totalPasses',
        'accuratePassesPercentage',
        'aerialDuelsWonPercentage',
        'totalLongBalls',
        'accurateLongBallsPercentage',
        'interceptions',
        'tackles',
        'groundDuelsWon',
        'groundDuelsWonPercentage',
        'possessionLost',
      ],
      bundes: {
        appearances: [],
        matchesStarted: [],
        touches: [],
        totalPasses: [],
        accuratePassesPercentage: [],
        aerialDuelsWonPercentage: [],
        totalLongBalls: [],
        accurateLongBallsPercentage: [],
        interceptions: [],
        tackles: [],
        groundDuelsWon: [],
        groundDuelsWonPercentage: [],
        possessionLost: [],
      },
      chL: {
        appearances: [],
        matchesStarted: [],
        touches: [],
        totalPasses: [],
        accuratePassesPercentage: [],
        aerialDuelsWonPercentage: [],
        totalLongBalls: [],
        accurateLongBallsPercentage: [],
        interceptions: [],
        tackles: [],
        groundDuelsWon: [],
        groundDuelsWonPercentage: [],
        possessionLost: [],
      },
      overall: {
        appearances: [],
        matchesStarted: [],
        touches: [],
        totalPasses: [],
        accuratePassesPercentage: [],
        aerialDuelsWonPercentage: [],
        totalLongBalls: [],
        accurateLongBallsPercentage: [],
        interceptions: [],
        tackles: [],
        tacklesOn90: [],
        groundDuelsWon: [],
        groundDuelsWonPercentage: [],
        possessionLost: [],
      },
    }
  },
  computed: {
    overallWidth() {
      const rtn = {}
      const overallArr = Object.keys(this.overall)
      for (const statName of overallArr) {
        const statValues = this.overall[statName]
        const maxValue = Math.max(...statValues)
        const statValuesPersentage = statValues.map(value => {
          return Math.round(value / maxValue * 100)
        })
        rtn[statName] = statValuesPersentage
      }
      return rtn
    },
    overallNorm() {
      const leagueStatsArr = Object.keys(this.bundes)
      const res = []
      leagueStatsArr.forEach((statName, idx) => {
        // res.push(statName + idx)

      })
      return res
    }
  },
  async beforeMount() {
    await this.fetchSofaData()
    console.log(this.overallWidth)
  },
  methods: {
    // computeWidth() {
    //   const cols = document.querySelectorAll('.cols-comparison')
    //   // const valuesArr = []
    //   // console.log(cols)
    //   // проходимся по колонкам с данными, разбивая их на элементы массива
    //   cols.forEach(colComparison => {
    //     colComparison.childNodes.forEach(statValueNode => {
    //       if (statValueNode.classList.contains('stat-value')) {
    //         const range = statValueNode.childNodes[0]
    //         range.querySelector('.range-width').style.width = `${range.innerText}%`
    //         if (range.innerText.length > 2) {
    //           range.querySelector('.range-width').style.width = `${range.innerText}`
    //         }
    //         if (range.childNodes.length > 2) {
    //           const percentFromHtml = statValueNode.querySelector('.for-width').innerText.slice(2,4)
    //           range.querySelector('.range-width').style.width = `${percentFromHtml}%`
    //         }
    //         if (statValueNode.classList.contains('interc')) {
    //           const dataForInterc = +range.innerText * 50
    //           // console.log(dataForInterc);
    //           range.querySelector('.range-width').style.width = `${dataForInterc}%`
    //         }
    //         if (statValueNode.classList.contains('possessionL')) {
    //           const dataForPossessionL = +range.innerText * 10
    //           range.querySelector('.range-width').style.width = `${dataForPossessionL}%`
    //         }
    //       }
    //     })
    //   })
    // },
    async fetchSofaData() {
      try {
        for (const league of this.leagues) {
          for (const idSeason of this.premLeagueSeasonsSofa) {
            const response = await this.$axios.get(`/api/v1/player/${this.rudigerId}/unique-tournament/${league}/season/${idSeason}/statistics/overall`)
            const stats = response.data.statistics

            this.bundes.appearances.push(stats.appearances)
            this.bundes.matchesStarted.push(stats.matchesStarted)
            this.bundes.touches.push(stats.touches)
            this.bundes.totalPasses.push(stats.totalPasses)
            this.bundes.totalLongBalls.push(stats.totalLongBalls)
            this.bundes.accuratePassesPercentage.push(stats.accuratePassesPercentage)
            this.bundes.aerialDuelsWonPercentage.push(stats.aerialDuelsWonPercentage)
            this.bundes.accurateLongBallsPercentage.push(stats.accurateLongBallsPercentage)
            this.bundes.interceptions.push(stats.interceptions)
            this.bundes.tackles.push(stats.tackles)
            this.bundes.groundDuelsWon.push(stats.groundDuelsWon)
            this.bundes.groundDuelsWonPercentage.push(stats.groundDuelsWonPercentage)
            this.bundes.possessionLost.push(stats.possessionLost)
          }
        }

        for (const idSeason of this.premLeagueSeasonsSofa) {
          const response = await this.$axios.get(`/api/v1/player/${this.rudigerId}/unique-tournament/${this.premLeague}/season/${idSeason}/statistics/overall`)
          const stats = response.data.statistics
          this.teams.push(response.data.team.shortName)
          this.bundes.appearances.push(stats.appearances)
          this.bundes.matchesStarted.push(stats.matchesStarted)
          this.bundes.touches.push(stats.touches)
          this.bundes.totalPasses.push(stats.totalPasses)
          this.bundes.totalLongBalls.push(stats.totalLongBalls)
          this.bundes.accuratePassesPercentage.push(stats.accuratePassesPercentage)
          this.bundes.aerialDuelsWonPercentage.push(stats.aerialDuelsWonPercentage)
          this.bundes.accurateLongBallsPercentage.push(stats.accurateLongBallsPercentage)
          this.bundes.interceptions.push(stats.interceptions)
          this.bundes.tackles.push(stats.tackles)
          this.bundes.groundDuelsWon.push(stats.groundDuelsWon)
          this.bundes.groundDuelsWonPercentage.push(stats.groundDuelsWonPercentage)
          this.bundes.possessionLost.push(stats.possessionLost)
        }
        for (const idSeason of this.chLeagueSeasonsSofa) {
          const response = await this.$axios.get(`/api/v1/player/${this.rudigerId}/unique-tournament/${this.chLeagueIdSofa}/season/${idSeason}/statistics/overall`)
          const stats = response.data.statistics
          this.chL.appearances.push(stats.appearances)
          this.chL.matchesStarted.push(stats.matchesStarted)
          this.chL.touches.push(stats.touches)
          this.chL.totalPasses.push(stats.totalPasses)
          this.chL.totalLongBalls.push(stats.totalLongBalls)
          this.chL.accuratePassesPercentage.push(stats.accuratePassesPercentage)
          this.chL.aerialDuelsWonPercentage.push(stats.aerialDuelsWonPercentage)
          this.chL.accurateLongBallsPercentage.push(stats.accurateLongBallsPercentage)
          this.chL.interceptions.push(stats.interceptions)
          this.chL.tackles.push(stats.tackles)
          this.chL.groundDuelsWon.push(stats.groundDuelsWon)
          this.chL.groundDuelsWonPercentage.push(stats.groundDuelsWonPercentage)
          this.chL.possessionLost.push(stats.possessionLost)
        }
        this.doOverall()
      } catch (e) {
        console.error(e)
      }
    },
    async fetchSeasonsTournament () {
      // по id игрока получаем id сезона в конкретной лиге. Зачастую нас будет интересовать нац лига, т.е. первая в списке
      // (возможно, придется добавлять условие для того, какую лигу я ожидаю увидеть, и вставлять название лиги в проверку, передавая через параметры)
      try {
        const response = await this.$axios.get(`https://api.sofascore.com/api/v1/player/${this.playerId}/statistics/seasons`)
        const firstTournament = response.data.uniqueTournamentSeasons[0]
        const allSeasons = firstTournament.seasons
        if (allSeasons.length > 3) {
          for (let i = 0; i < 4; i++) {
            this.seasonTournamentId.push(allSeasons[i].id)
            this.seasonTournamentYears.push(allSeasons[i].year)
          }
        } else {
          allSeasons.forEach(season => {
            this.seasonTournamentId.push(season.id)
            this.seasonTournamentYears.push(season.year)
          })
        }
      } catch (e) {
        console.error(e)
      }
    },
    doOverall() {
      this.overall.appearances = this.bundes.appearances.map((val, idx) => val + this.chL.appearances[idx])
      this.overall.matchesStarted = this.bundes.matchesStarted.map((val, idx) => val + this.chL.matchesStarted[idx])
      this.overall.touches = this.bundes.touches.map((val, idx) => Math.round((val / this.bundes.appearances[idx] + this.chL.touches[idx] / this.chL.appearances[idx]) / 2))
      this.overall.totalPasses = this.bundes.totalPasses.map((totalPasses, idx) => Math.round((totalPasses / this.bundes.appearances[idx] + this.chL.totalPasses[idx] / this.chL.appearances[idx]) / 2))
      this.overall.totalLongBalls = this.bundes.totalLongBalls.map((totalLongBalls, idx) => ((totalLongBalls / this.bundes.appearances[idx] + this.chL.totalLongBalls[idx] / this.chL.appearances[idx]) / 2).toFixed(1))
      this.overall.accuratePassesPercentage = this.bundes.accuratePassesPercentage.map((val, idx) => Math.round((val + this.chL.accuratePassesPercentage[idx]) / 2))
      this.overall.aerialDuelsWonPercentage = this.bundes.aerialDuelsWonPercentage.map((val, idx) => Math.round((val + this.chL.aerialDuelsWonPercentage[idx]) / 2))
      this.overall.accurateLongBallsPercentage = this.bundes.accurateLongBallsPercentage.map((val, idx) => Math.round((val + this.chL.accurateLongBallsPercentage[idx]) / 2))
      this.overall.interceptions = this.bundes.interceptions.map((interceptions, idx) => ((interceptions / this.bundes.appearances[idx] + this.chL.interceptions[idx] / this.chL.appearances[idx]) / 2).toFixed(1))
      this.overall.tackles = this.bundes.tackles.map((tackles, idx) => +(tackles + this.chL.tackles[idx]))
      this.overall.tacklesOn90 = this.bundes.tackles.map((tackles, idx) => ((tackles / this.bundes.appearances[idx] + this.chL.tackles[idx] / this.chL.appearances[idx]) / 2).toFixed(1))
      this.overall.groundDuelsWon = this.bundes.groundDuelsWon.map((groundDuelsWon, idx) => ((groundDuelsWon / this.bundes.appearances[idx] + this.chL.groundDuelsWon[idx] / this.chL.appearances[idx]) / 2).toFixed(1))
      this.overall.groundDuelsWonPercentage = this.bundes.groundDuelsWonPercentage.map((val, idx) => Math.round((val + this.chL.groundDuelsWonPercentage[idx]) / 2))
      this.overall.possessionLost = this.bundes.possessionLost.map((possessionLost, idx) => ((possessionLost / this.bundes.appearances[idx] + this.chL.possessionLost[idx] / this.chL.appearances[idx]) / 2).toFixed(1))
    },
  }
}
</script>

<style lang="scss">
$green: #07d0186e;
$yellow: #faf20182;
$orange: #fa8e01c9;
$red: #fa2601b6;

.container-fluid {
  &::before {
    content: '';
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    z-index: 0;
    background: rgba($color: #222, $alpha: .7);
    position: absolute;
  }
}
.stats-table {
  max-width: 950px;
  background-image: url('https://static.ua-football.com/img/upload/20/2942de.jpeg');
  background-repeat: no-repeat;
  background-size: contain;
  background-position: 50% 30%;
  color: white;
  position: relative;
  z-index: 1;
  .cols-comparison {
    margin-bottom: 15px;
  }
  .stat-legend {
    min-height: 110px;
    font-size: 16px;
    color: rgba($color: #fff, $alpha: .85);
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .stat-value {
    position: relative;
    text-align: left;
    padding-left: 5px;
    font-size: 16px;
    .range {
      &-width {
        width: 100%;
        height: 80%;
        display: block;
        background-color: #07d0186e;
        border-radius: 8px;
        position: absolute;
        z-index: -1;
        transform: translate(-5px, -93%);
      }
    }
  }
  .matches, .touches {
    // max-width: 62px;
  }
}
</style>
