<template lang="pug">
  .container-fluid.px-10.mx-auto
    .min-h-screen.flex.items-center.justify-center
      input.absolute.top-5.left-5.bg-gray-500.text-white.rounded-xl.py-2.px-3(class="focus:bg-gray-400 focus:outline-none focus:ring-1 focus:ring-blue-400 focus:border-transparent", type="text", v-model="playerId")
      button.absolute.top-7.right-5.button.inline-block.px-5.bg-blue-600.text-white.rounded-xl(@click="fetchSeasonsTournament") Get data by player ID
      .stats-table.text-center.font-sans.text-sm.grid.grid-cols-6(v-if="playerId && seasonTournamentIds.length > 0")
        .season
          .season.mx-1.py-3.stat-legend Сезоны
          .season.mx-1.py-3(v-for="(season, idx) in seasonTournamentYears[0]", :key="idx") {{ season }}
        .cols-comparison(v-if="isLoaded", v-for="(value, statName) in statNames" :key="value")
          .stat-legend.possessionL.mx-1.py-3 {{ value }}
          .stat-value.possessionL.mx-1.py-3(v-for="(seasonStats, idx) in overallNorm" :key="idx")
            span.range {{ statName.includes('Percentage') ? `${seasonStats[statName]} %` : seasonStats[statName] }}
              span.range-width(v-if="Object.keys(overallWidth).length > 0", :style="{ width: overallWidth[statName][idx] + '%' }")
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
      isLoaded: false,
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
      twoArrayWithStats: [ [], [] ],
      statsForBlocksWidth: [ {}, {}
        // { appearances: [52, 22, 48, 32] ...},
        // { appearances: [52, 22, 48, 32] ...},
      ],
    }
  },

  computed: {
    firstTournamentWithStats() {
      return this.twoArrayWithStats[0]
    },
    secondTournamentWithStats() {
      return this.twoArrayWithStats[1]
    },
    overallNorm() {
      const rtn = []
      for (let i = 0; i < this.firstTournamentWithStats.length; i++) {
        const objWithStats = {}
        const season = this.firstTournamentWithStats[i]
        for (const statName in season) {
          // среднее значение показателей в процентах, очевидно, высчитывать не нужно
          if (statName.includes('Percentage')) {
            // const meanValue = ((this.firstTournamentWithStats[i][statName] + this.secondTournamentWithStats[i][statName]) / 2).toFixed(1)
            const meanValue = (this.firstTournamentWithStats[i][statName]).toFixed(1)
            objWithStats[statName] = meanValue
          // среднее значение появлений в старте тоже
          } else if (statName === 'matchesStarted' || statName === 'appearances') {
            const value = Math.round(this.firstTournamentWithStats[i][statName] + this.secondTournamentWithStats[i][statName])
            objWithStats[statName] = value
          } else {
            const meanValue = ((this.firstTournamentWithStats[i][statName] + this.secondTournamentWithStats[i][statName]) / (this.firstTournamentWithStats[i].appearances + this.secondTournamentWithStats[i].appearances)).toFixed(2)
            objWithStats[statName] = meanValue
          }
        }
        console.log(this.overallWidth)
        console.log(this.statsForBlocksWidth)
        rtn.push(objWithStats)

      }
      return rtn
    },
    overallWidth() {
      const rtn = {}
      console.log('this.statsForBlocksWidth')
      console.log(this.statsForBlocksWidth)
      for (const statName in this.statsForBlocksWidth[0]) {
        console.log(this.statsForBlocksWidth)
        const statValues = []
        for (let i = 0; i < this.statsForBlocksWidth[0][statName].length; i++) {
          if (statName.includes('Percentage')) {
            // statValues.push((this.statsForBlocksWidth[0][statName][i] + this.statsForBlocksWidth[1][statName][i]) / 2)
            statValues.push(this.statsForBlocksWidth[0][statName][i])
          } else if (statName === 'matchesStarted' || statName === 'appearances') {
            statValues.push(this.statsForBlocksWidth[0][statName][i] + this.statsForBlocksWidth[1][statName][i])
          } else {
            statValues.push((this.statsForBlocksWidth[0][statName][i] + this.statsForBlocksWidth[1][statName][i]) / (this.statsForBlocksWidth[0].appearances[i] + this.statsForBlocksWidth[1].appearances[i]))
          }
        }

        console.log(this.statsForBlocksWidth[0].appearances)
        const maxValue = Math.max(...statValues)
        console.log(maxValue)
        const statValuesPersentage = statValues.map(value => {
          return Math.round(value / maxValue * 100)
        })
        console.log(statValuesPersentage)
        rtn[statName] = statValuesPersentage
      }
      return rtn
    },
  },
  watch: {
    async seasonTournamentIds() {
      if (this.seasonTournamentIds.length > 0) {
        for (const [idx, tournamentId] of this.firstAndSecondTournamentsId.entries()) {
          for (const seasonId of this.seasonTournamentIds[idx]) {
            const response = await this.$axios.get(`/api/v1/player/${this.playerId}/unique-tournament/${tournamentId}/season/${seasonId}/statistics/overall`)
            const statsResponce = response.data.statistics
            console.log(statsResponce)
            // данные для длины блоков
            for (const statName in this.statNames) {
              if (!Array.isArray(this.statsForBlocksWidth[idx][statName])) {
                this.statsForBlocksWidth[idx][statName] = []
              }
              this.statsForBlocksWidth[idx][statName].push(Math.round(statsResponce[statName]))
            }
            const objWithStats = {}
            for (const statName in this.statNames) {
              if (Object.hasOwnProperty.call(this.statNames, statName)) {
                objWithStats[statName] = Number(statsResponce[statName])
              }
            }
            this.twoArrayWithStats[idx].push(objWithStats)
          }
        }

        this.isLoaded = true
      }
    }
  },
  methods: {
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
