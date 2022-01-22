<template lang="pug">
  .container-fluid.px-10.mx-auto
    .min-h-screen.flex.items-center.justify-center
      input.absolute.top-5.left-5.bg-gray-500.text-white.rounded-xl.py-2.px-3(class="focus:bg-gray-400 focus:outline-none focus:ring-1 focus:ring-blue-400 focus:border-transparent", type="text", v-model="playerId")
      button.absolute.top-7.right-5.button.inline-block.px-5.bg-blue-600.text-white.rounded-xl(@click="fetchSeasonsTournament") Get data by player ID
      .stats-table.text-center.font-sans.text-sm.grid.grid-cols-6(v-if="playerId && seasonTournamentIds.length > 0")
        .season
          .season.mx-1.py-3.stat-legend Сезоны
          .season.mx-1.py-3(v-for="(season, idx) in seasonTournamentYears[0]", :key="idx") {{ season }}
        //- .xG
        //- .xA
        //- .teams
        //-   .team.mx-1.py-3.stat-legend Команда
        //-   .team.mx-1.py-3(v-for="(team, idx) in teams", :key="idx") {{ team }}
        //- .matches
        //-   .match.mx-1.py-3.stat-legend Матчей в сезоне
        //-   .match.mx-1.py-3(v-for="(appearance, idx) in overallNorm.appearances" :key="idx") {{ appearance }}
        //- .matches
        //-   .match.mx-1.py-3.stat-legend Из них в старте
        //-   .match.mx-1.py-3(v-for="(matchesInSeason, idx) in overallNorm.matchesStarted" :key="idx") {{ matchesInSeason }}
        //- .cols-comparison.touches
        //-   .stat-legend.touch.mx-1.py-3 Касаний / 90мин
        //-   .stat-value.touch.mx-1.py-3(v-for="(touches, idx) in overall.touches" :key="idx")
        //-     span.range {{ touches }}
        //-       span.range-width(:style="{ width: overallWidth.touches[idx] + '%' }")
        //- .cols-comparison.totalPasses
        //-   .stat-legend.accurateP.mx-1.py-3 Пасов / 90мин
        //-   .stat-value.accurateP.mx-1.py-3(v-for="(totalPasses, idx) in overall.totalPasses" :key="idx")
        //-     span.range {{ totalPasses }}
        //-       span.range-width(:style="{ width: overallWidth.totalPasses[idx] + '%' }")
        //- .cols-comparison.accuratePassesPercentage
        //-   .stat-legend.accuratePP.mx-1.py-3 % успешных пасов / 90мин
        //-   .stat-value.accuratePP.mx-1.py-3(v-for="(accuratePassesPercentage, idx) in overall.accuratePassesPercentage" :key="idx")
        //-     span.range {{ accuratePassesPercentage }}%
        //-       span.range-width(:style="{ width: overallWidth.accuratePassesPercentage[idx] + '%' }")
        //- .cols-comparison.aerialDuelsWonPercentage
        //-   .stat-legend.aerialDWP.mx-1.py-3 % выигранных воздушных единоборств
        //-   .stat-value.aerialDWP.mx-1.py-3(v-for="(aerialDuelsWonPercentage, idx) in overall.aerialDuelsWonPercentage" :key="idx")
        //-     span.range {{ aerialDuelsWonPercentage }}%
        //-       span.range-width(:style="{ width: overallWidth.aerialDuelsWonPercentage[idx] + '%' }")
        //- .cols-comparison.groundDuelsWon
        //-   .stat-legend.groundDW.mx-1.py-3 Выигранных наземных дуэлей
        //-   .stat-value.groundDW.mx-1.py-3(v-for="(groundDuelsWon, idx) in overall.groundDuelsWon" :key="idx")
        //-     span.range {{ groundDuelsWon }}
        //-       span.range-width(:style="{ width: overallWidth.groundDuelsWon[idx] + '%' }")
        //- .cols-comparison.groundDuelsWonPercentage
        //-   .stat-legend.groundDW.mx-1.py-3 % успешных наземных дуэлей
        //-   .stat-value.groundDW.mx-1.py-3(v-for="(groundDuelsWonPercentage, idx) in overall.groundDuelsWonPercentage" :key="idx")
        //-     span.range {{ groundDuelsWonPercentage }}%
        //-       span.range-width(:style="{ width: overallWidth.groundDuelsWonPercentage[idx] + '%' }")
        //- .cols-comparison.longBalls
        //-   .stat-legend.lB.mx-1.py-3 Длинных передачи / 90мин
        //-   .stat-value.lB.mx-1.py-3(v-for="(totalLongBalls, idx) in overall.totalLongBalls" :key="idx")
        //-     span.range {{ totalLongBalls }}
        //-       span.range-width(:style="{ width: overallWidth.totalLongBalls[idx] + '%' }")
        //- .cols-comparison.accurateLongBallsPercentage
        //-   .stat-legend.lB.mx-1.py-3 % успешных длинных передач / 90мин
        //-   .stat-value.lB.mx-1.py-3(v-for="(accurateLongBallsPercentage, idx) in overall.accurateLongBallsPercentage" :key="idx")
        //-     span.range {{ accurateLongBallsPercentage }}%
        //-       span.range-width(:style="{ width: overallWidth.accurateLongBallsPercentage[idx] + '%' }")
        //- .cols-comparison.interceptions
        //-   .stat-legend.interc.mx-1.py-3 Перехватов / 90мин
        //-   .stat-value.interc.mx-1.py-3(v-for="(interceptions, idx) in overall.interceptions" :key="idx")
        //-     span.range {{ interceptions }}
        //-       span.range-width(:style="{ width: overallWidth.interceptions[idx] + '%' }")
        //- .cols-comparison.tackles
        //-   .stat-legend.accurateP.mx-1.py-3 Отборов за игру
        //-   .stat-value.accurateP.mx-1.py-3(v-for="(tackles, idx) in overall.tackles" :key="idx")
        //-     span.range {{ +tackles.toFixed(1) }}
        //-       span.range-width(:style="{ width: overallWidth.tackles[idx] + '%' }")
        //- //- .cols-comparison.tacklesOn90
        //- //-   .stat-legend.accurateP.mx-1.py-3 % успешных отборов за игру
        //- //-   .stat-value.accurateP.mx-1.py-3(v-for="(tacklesOn90, idx) in overall.tacklesOn90" :key="idx")
        //- //-     span.range {{ tacklesOn90 }}%
        //- //-       span.range-width(:style="{ width: overallWidth.tacklesOn90[idx] + '%' }")
        //- .cols-comparison.possessionLost
        //-   .stat-legend.possessionL.mx-1.py-3 Утрачено владение / 90мин
        //-   .stat-value.possessionL.mx-1.py-3(v-for="(possessionLost, idx) in overall.possessionLost" :key="idx")
        //-     span.range {{ possessionLost }}
        //-       span.range-width(:style="{ width: overallWidth.possessionLost[idx] + '%' }")

        //- .cols-comparison(v-if="overallNorm && overallNorm.length > 0", v-for="(value, statName) in statNames" :key="value")
        //-   .stat-legend.possessionL.mx-1.py-3 {{ value }}
        //-   .stat-value.possessionL.mx-1.py-3(v-for="(seasonStats, idx) in overallNorm" :key="idx")
        //-     span.range {{ seasonStats[statName].toFixed(1) }}
        //-       span.range-width(:style="{ width: overallWidth.possessionLost[idx] + '%' }")
</template>

<script>
export default {
  name: 'IndexPage',
  data () {
    return {
      playerId: localStorage.getItem('playerId') ?? '',
      seasonTournamentIds: [ [], [] ],
      seasonTournamentYears: [ [], [] ],
      firstAndSecondTournamentsId: [],
      // seasonsItem: ['2019/2020', '2020/2021', '2021/2022'],
      // // seasonsItem: ['2018/2019', '2019/2020', '2020/2021', '2021/2022'],
      // years: [2018, 2019, 2020, 2021],
      // // sofascore. Take an average values in this tournament
      // bundesIdSofa: 35,
      // bundesSeasonsSofa: [17597, 23538, 28210, 37166],
      // premLeague: 17,
      // // premLeagueSeasonsSofa: [17359, 23776, 29415, 37036],
      // premLeagueSeasonsSofa: [23776, 29415, 37036],
      // chLeagueIdSofa: 7,
      // // chLeagueSeasonsSofa: [17351, 23766, 29267, 36886], for Sule
      // chLeagueSeasonsSofa: [23766, 29267, 36886],
      // suleId: 168937,
      // rudigerId: 142622,

      // sofa stats
      teams: [],
      // leagues: [
      //   {
      //     17: [17359, 23776, 29415, 37036]
      //   },
      //   {
      //     7: [23766, 29267, 36886]
      //   }
      // ],
      statNames: {
        appearances: 'Матчей в сезоне',
        matchesStarted: 'Из них в старте',
        touches: 'Касаний за игру',
        totalPasses: 'Пасов за игру',
        accuratePassesPercentage: '% успешных пасов за игру',
        aerialDuelsWonPercentage: '% выигранных воздушных единоборств за игру',
        totalLongBalls: 'Длинных передачи за игру',
        accurateLongBallsPercentage: '% успешных длинных передач за игру',
        interceptions: 'Перехватов за игру',
        tackles: 'Отборов за игру',
        groundDuelsWon: 'Выигранных наземных дуэлей',
        groundDuelsWonPercentage: '% успешных наземных дуэлей',
        possessionLost: 'Утрачено владение за игру',
      },
      // twoArrayWithStats: [ {app:33, touch:73}, {app:21, touch:84}... ]
      twoArrayWithStats: [ [], [] ],
    }
  },

  computed: {
    overallWidth() {
      const rtn = {}
      const overallArr = Object.keys(this.statNames)
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
    firstTournamentWithStats() {
      return this.twoArrayWithStats[0]
    },
    secondTournamentWithStats() {
      return this.twoArrayWithStats[1]
    },
    overallNorm() {
      const rtn = []
      console.log(rtn)
      const objWithStats = {}
      for (let i = 0; i < this.firstTournamentWithStats.length; i++) {
        const season = this.firstTournamentWithStats[i]
        for (const statName in season) {
          // среднее значение показателей в процентах, очевидно, высчитывать не нужно
          if (statName.includes('Percentage')) {
            const meanValue = ((this.firstTournamentWithStats[i][statName] + this.secondTournamentWithStats[i][statName]) / 2).toFixed(1)
            objWithStats[statName] = meanValue
          // среднее значение появлений в старте тоже
          } else if (statName === 'matchesStarted' || statName === 'appearances') {
            const value = this.firstTournamentWithStats[i][statName] + this.secondTournamentWithStats[i][statName]
            objWithStats[statName] = value
          } else {
            const meanValue = ((this.firstTournamentWithStats[i][statName] + this.secondTournamentWithStats[i][statName]) / (this.firstTournamentWithStats[i].appearances + this.secondTournamentWithStats[i].appearances)).toFixed(2)
            objWithStats[statName] = meanValue
          }
        }
        rtn.push(objWithStats)
        console.log(i)
        console.log(objWithStats);
        console.log(rtn)

      }
      return rtn
    }
  },
  watch: {
    async seasonTournamentIds() {
      if (this.seasonTournamentIds.length > 0) {
        for (const [idx, tournamentId] of this.firstAndSecondTournamentsId.entries()) {
          for (const seasonId of this.seasonTournamentIds[idx]) {
            const response = await this.$axios.get(`/api/v1/player/${this.playerId}/unique-tournament/${tournamentId}/season/${seasonId}/statistics/overall`)
            const statsResponce = response.data.statistics
            // objWithStats для будущего { app:33, touch:73 }
            const objWithStats = {}
            for (const statName in this.statNames) {
              if (Object.hasOwnProperty.call(this.statNames, statName)) {
                // цель: в overallEPL добавляем объекты с ключом-значением
                // overallEPL[0][appearences] = 33
                // делаем что-то вроде
                // { app:33, touch:73 }
                objWithStats[statName] = Number(statsResponce[statName])
              }
            }
            this.twoArrayWithStats[idx].push(objWithStats)
          }
        }
      }
      console.log(this.overallNorm)
    }
  },
  // async beforeMount() {
  //   console.log('a');
  //   await this.fetchSofaData()
  // },
  methods: {
    // async fetchSofaData() {
    //   try {
    //     console.log('b');
    //     for (const seasonId of this.seasonTournamentIds) {
    //       const response = await this.$axios.get(`/api/v1/player/${this.playerId}/unique-tournament/${this.uniqueTournament}/season/${seasonId}/statistics/overall`)
    //       const stats = response.data.statistics

    //       this.bundes.appearances.push(stats.appearances)
    //       this.bundes.matchesStarted.push(stats.matchesStarted)
    //       this.bundes.touches.push(stats.touches)
    //       this.bundes.totalPasses.push(stats.totalPasses)
    //       this.bundes.totalLongBalls.push(stats.totalLongBalls)
    //       this.bundes.accuratePassesPercentage.push(stats.accuratePassesPercentage)
    //       this.bundes.aerialDuelsWonPercentage.push(stats.aerialDuelsWonPercentage)
    //       this.bundes.accurateLongBallsPercentage.push(stats.accurateLongBallsPercentage)
    //       this.bundes.interceptions.push(stats.interceptions)
    //       this.bundes.tackles.push(stats.tackles)
    //       this.bundes.groundDuelsWon.push(stats.groundDuelsWon)
    //       this.bundes.groundDuelsWonPercentage.push(stats.groundDuelsWonPercentage)
    //       this.bundes.possessionLost.push(stats.possessionLost)
    //     }
    //     this.doOverall()
    //   } catch (e) {
    //     console.error(e)
    //   }
    // },
    async fetchSeasonsTournament() {
      // по id игрока получаем id сезона в конкретной лиге. Зачастую нас будет интересовать нац лига, т.е. первая в списке
      // (возможно, придется добавлять условие для того, какую лигу я ожидаю увидеть, и вставлять название лиги в проверку, передавая через параметры)
      localStorage.setItem('playerId', this.playerId)
      try {
        const response = await this.$axios.get(`https://api.sofascore.com/api/v1/player/${this.playerId}/statistics/seasons`)
        // перебираем все турниры и ищет нац лигу и ЛЧ
        let succesTournament = 0
        for (let i = 0; i < response.data.uniqueTournamentSeasons.length; i++) {
          // смотрим только первые два турнира - как правило это будут нац лига и ЛЧ / ЛЕ
          const tournamentName = response.data.uniqueTournamentSeasons[i].uniqueTournament.name
          console.log(tournamentName)
          if (tournamentName === 'Premier League' || tournamentName === 'UEFA Champions League' || tournamentName === 'UEFA Europa League') {
            this.firstAndSecondTournamentsId.push(response.data.uniqueTournamentSeasons[i].uniqueTournament.id)
            const allSeasons = response.data.uniqueTournamentSeasons[i].seasons
            if (allSeasons.length > 3) {
              for (let k = 0; k < 4; k++) {
                this.seasonTournamentIds[succesTournament].push(allSeasons[k].id)
                this.seasonTournamentYears[succesTournament].push(allSeasons[k].year)
              }
            } else {
              allSeasons.forEach(season => {
                this.seasonTournamentIds[succesTournament].push(season.id)
                this.seasonTournamentYears[succesTournament].push(season.year)
              })
            }
            succesTournament += 1
            if (succesTournament === 2) {
              break
            }
          }
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
  background-image: url('https://static.ua-football.com/img/upload/20/2942de.jpeg');
  background-repeat: no-repeat;
  background-size: cover;
  background-position: 50% 30%;

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
  color: white;
  position: relative;
  z-index: 1;
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
