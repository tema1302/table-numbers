<template lang="pug">
  .component
    button.relative.button.inline-block.px-5.bg-blue-600.text-white.rounded-xl(@click="fetchSeasonsTournament") Get data
    .stats-table.text-left.text-sm.grid.grid-cols-1(v-if="playerId && seasonTournamentIds")
      h4.text-white.text-xl.col-span-1 {{ playerName }}
      //- .season
      //-   .season.mx-1.py-3.stat-legend Сезоны
      //-   .season.mx-1.py-2(v-for="(season, idx) in seasonTournamentYears", :key="idx") {{ season }}
      //- .season
      //-   .season.mx-1.py-3.stat-legend Сезоны
      //-   .season.mx-1.py-2(v-for="(season, idx) in seasonTournamentYears", :key="idx") {{ season }}
      //- .season
      //-   .season.mx-1.py-3.stat-legend Сезоны
      //-   .season.mx-1.py-2(v-for="(season, idx) in seasonTournamentYears", :key="idx") {{ season }}
      //- .season
      //-   .season.mx-1.py-3.stat-legend Сезоны
      //-   .season.mx-1.py-2(v-for="(season, idx) in seasonTournamentYears", :key="idx") {{ season }}

      //- .cols-comparison(v-if="isLoaded", v-for="(russStatName, statName, idx) in statNames" :key="statName")
      //-   .stat-legend.mx-1.relative
      //-     .stat-legend-text.absolute {{ russStatName }} / {{ statName }}
      //-   .stat-value.mx-1.py-2(v-for="(val2, i) in overallStatByMatch[statName]" :key="val2")
      //-     span.range {{ statName.includes('Percentage') ? `${val2} %` : val2 }}
      .cols-comparison(v-if="isLoaded", v-for="(russStatName, statName, idx) in statNames" :key="statName")
        .stat-legend.mx-1.relative
          .stat-legend-text.absolute {{ russStatName }} / {{ statName }}
        .stat-value.mx-1.py-2(v-for="(value, i) in overallStatWithCombinedIndicators[statName]" :key="value")
          span.range {{ statName.includes('Percentage') ? `${value} %` : value }}
            span.range-width(:style="{ width: overallWidth[statName][i] + '%' }")
</template>

<script>
export default {
  name: 'PlayerFirst',
  props: {
    playerId: {
      type: Number,
      default: 0,
    },
    playerName: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      // playerId: localStorage.getItem('playerId') ?? '',
      seasonTournamentIds: [],
      seasonTournamentYears: [],
      tournamentsIds: [],
      isLoaded: false,
      // needSeasons: ['21/22', '20/21', '19/20', '18/19'],
      // needSeasons: ['21/22', '20/21', '19/20'],
      needSeasons: ['21/22', '20/21'],
      // needSeasons: ['21/22'],
      statNames: {
        appearances: 'Матчей в сезоне',

        matchesStarted: 'Выходов в старте',
        touches: 'Касаний (действий с мячом) за игру',
        totalPasses: 'Пасов за игру',
        accuratePasses: 'Успешных передач за игру',
        accurateFinalThirdPasses: 'Успешных передач в финальной трети',
        totalOppositionHalfPasses: 'Пасов на чужой половине поля за игру',

        // bigChancesCreated: 'Явных голевых моментов создано за игру',
        // assists: 'Ассистов за игру',

        // successfulDribbles: 'Успешного дриблинга за игру',
        // successfulDribblesPercentage: '% успешного дриблинга',

        // possessionWonAttThird: 'Возвратов владения (финальная треть) за игру',

        // totalShots: 'Ударов за игру',

        // shotsFromOutsideTheBox: 'Ударов за пределами штрафной за игру',
        // shotsOnTarget: 'Ударов по воротам за игру',
        // shotsOffTarget: 'Ударов мимо ворот за игру',

        // bigChancesMissed: 'Упущено голевых моментов за игру',
        // goals: 'Всего голов за игру',
        // penaltyGoals: 'Голов из пенальти за игру',
        // goalsAssistsSum: 'Гол + пас',
        // goalConversionPercentage: 'Конвертация ударов в голы в %',

        // totalLongBalls: 'Длинных передачи за игру',
        // accurateLongBallsPercentage: '% успешных длинных передач',
        // accurateChippedPasses: 'Успешных пасов с подсечкой',
        // keyPasses: 'Ключевых передач за игру',

        // aerialDuelsWon: 'Выигранных воздушных единоборств за игру',
        // aerialDuelsWonPercentage: '% выигранных воздушных единоборств',

        // groundDuelsWon: 'Выигранных наземных дуэлей за игру',
        // groundDuelsWonPercentage: '% выигранных наземных дуэлей',
        // possessionLost: 'Потерей мяча за игру',

        // на всякий случай

        // interceptions: 'Перехватов за игру',
        // tackles: 'Отборов за игру',
        // clearances: 'Выносов за игру',
        // blockedShots: 'Заблокировано ударов за игру',
        // offsides: 'Офсайдов',

        // cleanSheet: 'Сухарей',
      },
      tableNumbers: {},
      statsForBlocksWidth: {},
      overallStatByMatch: {},
      smallerIndicators: [
        'matchesStarted',
        // 'totalOppositionHalfPasses',
        'accurateFinalThirdPasses',
        'accuratePasses',
      ],
      biggerIndicators: [
        'appearances',
        // 'totalPasses',
        'totalPasses',
        'totalPasses',
      ],
      combinedIndicatorsName: {
        matchesStarted: 'Матчей в старте',
        // totalOppositionHalfPasses: 'Пасов на чужой половине поля за игру',
        accurateFinalThirdPasses: 'Успешных передач в финальной трети',
        accuratePasses: 'Успешных передач за игру',
      },
      combinedIndicators: {},
      overallStatWithCombinedIndicators: {},

      overallWidth: {},
    }
  },

  // computed: {
  // },
  watch: {
    async seasonTournamentIds() {
      for (const [idx, seasonId] of this.seasonTournamentIds.entries()) {
        console.log(seasonId)
        const response = await this.$axios.get(
          `/api/v1/player/${this.playerId}/unique-tournament/${this.tournamentsIds[idx]}/season/${this.seasonTournamentIds[idx]}/statistics/overall`
        )
        const statsResponce = response.data.statistics
        // данные для длины блоков
        // {app: [], touches: [] ...}
        for (const statName in this.statNames) {
          if (!Array.isArray(this.tableNumbers[statName]))
            this.tableNumbers[statName] = []
          this.tableNumbers[statName].push(statsResponce[statName])
        }
      }
      console.log('this.isLoaded')

      this.isLoaded = true
      this.getStatByMatch()
      console.log(this.isLoaded)
    },
  },
  methods: {
    async fetchSeasonsTournament() {
      // по id игрока получаем id сезона в конкретной лиге. Зачастую нас будет интересовать нац лига, т.е. первая в списке
      // (возможно, придется добавлять условие для того, какую лигу я ожидаю увидеть, и вставлять название лиги в проверку, передавая через параметры)
      localStorage.setItem('playerId', this.playerId)
      try {
        const response = await this.$axios.get(
          `/api/v1/player/${this.playerId}/statistics/seasons`
        )
        let succesSeasons = 0
        while (succesSeasons < this.needSeasons.length) {
          for (
            let i = 0;
            i < response.data.uniqueTournamentSeasons.length;
            i++
          ) {
            const tournament = response.data.uniqueTournamentSeasons[i]
            const tournamentName = tournament.uniqueTournament.name
            if (
              tournamentName === 'Premier League' ||
              tournamentName === 'LaLiga' ||
              tournamentName === 'Ligue 1' ||
              tournamentName === 'Serie A' ||
              tournamentName === 'Bundesliga' ||
              tournamentName === 'Eredivisie'
            ) {
              // проверяем, есть ли в данном турнире нужный год (напр. Лукаку выступал в АПЛ в сезоне 21/22 - хотим выводить именно этот сезон в первую очередь)
              const hasNeedSeason = tournament.seasons.some(
                (season) => season.year === this.needSeasons[succesSeasons]
              )
              if (!hasNeedSeason) continue
              const needSeason = tournament.seasons.find(
                (season) => season.year === this.needSeasons[succesSeasons]
              )
              this.tournamentsIds.push(tournament.uniqueTournament.id)
              this.seasonTournamentIds.push(needSeason.id)
              this.seasonTournamentYears.push(needSeason.year)
              succesSeasons++
              if (succesSeasons === this.needSeasons.length) {
                i = response.data.uniqueTournamentSeasons.length
                break
              }
            }
          }
        }
      } catch (e) {
        console.error(e)
      }
    },

    getStatByMatch() {
      for (const statName in this.tableNumbers) {
        if (Object.hasOwnProperty.call(this.tableNumbers, statName)) {
          const arrStatValue = this.tableNumbers[statName]
          if (
            statName === 'matchesStarted' ||
            statName === 'appearances' ||
            statName === 'cleanSheet' ||
            // statName === 'assists' ||
            // statName === 'bigChancesCreated' ||
            // statName === 'penaltyGoals' ||
            // statName === 'goals' ||
            // statName === 'bigChancesMissed' ||
            statName === 'goalsAssistsSum'
          ) {
            this.overallStatByMatch[statName] = arrStatValue
          } else {
            const arrStatValueByMatch = arrStatValue.map((statValue, idx) => {
              if (statName.includes('Percentage')) {
                return +statValue.toFixed(1)
              } else {
                return +(
                  statValue / this.tableNumbers.appearances[idx]
                ).toFixed(1)
              }
            })
            // console.log(statName)
            // console.log(arrStatValueByMatch)
            this.overallStatByMatch[statName] = arrStatValueByMatch
          }
        }
      }
      console.log('getStatByMatch')
      console.log(this.overallStatByMatch)
      this.combineIndicatorsTotal()
    },
    combineIndicatorsTotal() {
      if (Object.keys(this.overallStatByMatch).length !== 0) {
        const statMatch = this.overallStatByMatch
        const arrEngNames = Object.keys(this.combinedIndicatorsName)
        for (const [idx, biggerVName] of this.biggerIndicators.entries()) {
          const smallerVName = this.smallerIndicators[idx]
          const indicatorName = arrEngNames[idx]

          const arrVal = []
          for (let i = 0; i < this.needSeasons.length; i++) {
            const biggerValue = statMatch[biggerVName][i]
            const smallerValue = statMatch[smallerVName][i]
            const percentage = ((smallerValue / biggerValue) * 100).toFixed(1)
            arrVal.push(`${smallerValue}/${biggerValue} (${percentage} %)`)
          }
          // this.overallStatByMatch[indicatorName] = arrVal
          this.combinedIndicators[indicatorName] = arrVal
        }
        console.log('combinedIndicators')
        console.log(this.combinedIndicators)
      }
      this.setOverallStatWithCombinedIndicators()
    },
    combineIndicatorsPercentage() {},
    // filterKeyDublicate() {
    //   for (const statName in this.overallStatByMatch) {
    //     if (Object.hasOwnProperty.call(this.overallStatByMatch, statName)) {
    //       const element = this.overallStatByMatch[statName];

    //     }
    //   }
    //   const startKeys = Object.keys(this.overallStatByMatch)
    //   const combinedKeys = Object.keys(this.combinedIndicators)
    //   this.setOverallStatWithCombinedIndicators()
    // },
    setOverallStatWithCombinedIndicators() {
      console.log('overallStatWithCombinedIndicators')
      this.overallStatWithCombinedIndicators = Object.assign(
        this.overallStatByMatch,
        this.combinedIndicators
      )
      console.log(this.overallStatWithCombinedIndicators)
      this.setOverallWidth()
    },
    setOverallWidth() {
      console.log('overallWidth')
      const overallArr = Object.keys(this.overallStatWithCombinedIndicators)
      for (const statName of overallArr) {
        let statValues = this.overallStatWithCombinedIndicators[statName]
        if (typeof statValues[0] === 'string') {
          statValues = statValues.map((val) => {
            console.log(val)
            if (val.slice(-7, -3) === '00.0') return +val.slice(-8, -3)
            console.log(+val.slice(-7, -3))
            return +val.slice(-7, -3)
          })
        }
        console.log(statValues)
        const maxValue = Math.max(...statValues)
        const statValuesPersentage = statValues.map((value) => {
          return Math.round((value / maxValue) * 100)
        })
        this.overallWidth[statName] = statValuesPersentage
      }
    },
  },
}
</script>

<style lang="scss">
.component:first-child .cols-comparison .stat-legend {
  opacity: 1;
}
.component .cols-comparison .stat-legend {
  opacity: 0;
}
.stats-table {
  width: 190px;
  padding: 15px;
  border: 1px solid rgba($color: #ffffff, $alpha: 0.1);
  margin: 70px 0 0;
  color: white;
  position: relative;
  z-index: 1;
  // скрываем матчи (хотя один раз нужно будет показать)
  .cols-comparison:first-of-type {
    display: none;
  }
  .cols-comparison:nth-child(2) .stat-value .range-width {
    background-color: #4e50d8c2;
  }
  .cols-comparison:nth-child(3) .stat-value .range-width {
    background-color: #5b2cac9f;
  }
  .cols-comparison:nth-child(4) .stat-value .range-width {
    background-color: #4595e195;
  }
  .cols-comparison:nth-child(5) .stat-value .range-width {
    background-color: #8184e6c6;
  }
  .cols-comparison:nth-child(6) .stat-value .range-width {
    background-color: #4e50d8c2;
  }
  .cols-comparison:nth-child(7) .stat-value .range-width {
    background-color: #4595e195;
  }
  .cols-comparison:nth-child(8) .stat-value .range-width {
    background-color: #8184e6c6;
  }
  .cols-comparison:nth-child(9) .stat-value .range-width {
    background-color: #4e50d8c2;
  }
  h4 {
    min-height: 60px;
  }
  .stat-legend {
    min-height: 50px;
    font-size: 13px;
    color: rgba($color: #fff, $alpha: 0.85);
    line-height: 1;
    &-text {
      width: 300%;
      // text-align: center;
      // left: 50%;
      // transform: translateX(-50%);
      bottom: 20%;
    }
  }
  .stat-value {
    position: relative;
    text-align: left;
    padding-left: 5px;
    font-size: 16px;
    font-family: 'Open Sans Condensed', sans-serif;
    .range {
      font-size: 18px;
      &-width {
        width: 100%;
        height: 75%;
        display: block;
        border-radius: 8px;
        position: absolute;
        z-index: -1;
        transform: translate(-5px, -93%);
      }
    }
  }
  .season {
    font-family: 'Open Sans Condensed', sans-serif;
    max-width: 62px;
    text-align: center;
    font-size: 18px;
    &.stat-legend {
      min-height: 50px;
      justify-content: center;
      opacity: 0;
    }
  }
}
</style>
