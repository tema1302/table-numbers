<template lang="pug">
  .component
    button.relative.button.inline-block.px-5.bg-blue-600.text-white.rounded-xl(@click="fetchSeasonsTournament") Get data
    .stats-table.text-left.text-sm.grid.grid-cols-1(v-if="playerId && seasonTournamentIds")
      h4.text-white.text-xl.col-span-1 {{ playerName }}
      .cols-comparison(v-if="isLoaded", v-for="(russStatName, statName, idx) in statNames" :key="statName")
        .stat-legend.mx-1.relative
          .stat-legend-text.absolute.tracking-wider {{ russStatName }} / {{ statName }}
        .stat-value.mx-1.py-2(v-for="(value, i) in overallStatWithCombinedIndicators[statName]" :key="value")
          .mx-1.text-xs {{ needSeasons[i] }}
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
      needSeasons: [
        '22/23',
        '21/22',
        '20/21',
        // '19/20',
        // '18/19',
        // '17/18',
        // '16/17',
        // '15/16',
      ],
      statNames: {
        minutesPlayed: 'Минут в сезоне',

        appearances: 'Матчей в сезоне',
        matchesStarted: 'Выходов в старте',
        touches: "Касаний (действий с мячом) на 90'",
        totalPasses: "Пасов на 90'",
        accuratePasses: "Успешных передач на 90'",
        accurateFinalThirdPasses: 'Успешных передач в финальной трети',
        totalOppositionHalfPasses: "Пасов на чужой половине поля на 90'",

        bigChancesCreated: "Явных голевых моментов создано на 90'",
        assists: "Ассистов на 90'",

        successfulDribbles: "Успешного дриблинга на 90'",
        successfulDribblesPercentage: '% успешного дриблинга',

        possessionWonAttThird: "Возвратов владения (финальная треть) на 90'",

        totalShots: "Ударов на 90'",

        shotsFromOutsideTheBox: "Ударов за пределами штрафной на 90'",
        shotsOnTarget: "Ударов по воротам на 90'",
        shotsOffTarget: "Ударов мимо ворот на 90'",

        bigChancesMissed: "Упущено голевых моментов на 90'",
        goals: "Всего голов на 90'",
        penaltyGoals: "Голов из пенальти на 90'",
        goalsAssistsSum: 'Гол + пас',
        goalConversionPercentage: 'Конвертация ударов в голы в %',

        totalLongBalls: "Длинных передачи на 90'",
        accurateLongBallsPercentage: '% успешных длинных передач',
        accurateChippedPasses: 'Успешных пасов с подсечкой',
        keyPasses: "Ключевых передач на 90'",

        dispossessed: "Лишен мяча без силового контакта на 90'",
        possessionLost: "Потерей мяча на 90'",

        aerialDuelsWon: "Выигранных воздушных единоборств на 90'",
        aerialDuelsWonPercentage: '% выигранных воздушных единоборств',

        groundDuelsWon: "Выигранных наземных дуэлей на 90'",
        groundDuelsWonPercentage: '% выигранных наземных дуэлей',

        // на всякий случай

        interceptions: "Перехватов на 90'",
        tackles: "Отборов на 90'",
        clearances: "Выносов на 90'",
        blockedShots: "Заблокировано ударов на 90'",
        // fouls: "Нарушений правил на 90'",
        // yellowCards: "Желтых карточек на 90'",

        // errorLeadToShot: 'Ошибок, приведших к удару',
        // errorLeadToGoal: 'Ошибок, приведших к голу',

        // offsides: 'Офсайдов',
        // cleanSheet: 'Сухарей',
      },
      tableNumbers: {},
      statsForBlocksWidth: {},
      overallStatByMatch: {},
      smallerIndicators: [
        'matchesStarted',
        'totalOppositionHalfPasses',
        'accurateFinalThirdPasses',
        'accuratePasses',
      ],
      biggerIndicators: [
        'appearances',
        'totalPasses',
        'totalPasses',
        'totalPasses',
      ],
      combinedIndicatorsName: {
        matchesStarted: 'Матчей в старте',
        totalOppositionHalfPasses: "Пасов на чужой половине поля на 90'",
        accurateFinalThirdPasses: 'Успешных передач в финальной трети',
        accuratePasses: "Успешных передач на 90'",
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
            statName === 'errorLeadToGoal' ||
            statName === 'errorLeadToShot' ||
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
                  (statValue / this.tableNumbers.minutesPlayed[idx]) *
                  90
                ).toFixed(2)
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
  width: 360px;
  padding: 12px 16px;
  border: 1px solid rgba($color: #ffffff, $alpha: 0.1);
  margin: 70px 0 0;
  color: white;
  position: relative;
  z-index: 1;
  // скрываем матчи (хотя один раз нужно будет показать)
  .cols-comparison:first-of-type {
    display: none;
  }

  $colors: #ff8256, #786fa6, #da628c, #3099a7, #303952, #c56cf0, #7158e2;
  @each $color in $colors {
    $i: index($colors, $color);
    .cols-comparison:nth-child(7n + #{$i}) .stat-value .range-width {
      background-color: $color;
    }
  }

  h4 {
    min-height: 60px;
    position: sticky;
    display: flex;
    align-items: center;
    text-align: center;
    justify-content: center;
    z-index: 10;
    top: 4px;
    background: rgba(
      255,
      255,
      255,
      0.2
    ); // Make sure this color has an opacity of less than 1
    backdrop-filter: blur(8px);
    border-radius: 8px;
    // -webkit-clip-path: polygon(53% 0, 100% 0, 100% 92%, 51% 100%, 0 92%, 0 0);
    // clip-path: polygon(53% 0, 100% 0, 100% 92%, 51% 100%, 0 92%, 0 0);
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
        height: 60%;
        display: block;
        border-radius: 8px;
        position: absolute;
        z-index: -1;
        transform: translate(-8px, -96%) skew(-21deg, 0deg);
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
