<template lang="pug">
  .component
    button.relative.button.inline-block.px-5.bg-blue-600.text-white.rounded-xl(@click="fetchMatchesIds") Get data
    .stats-table.text-left.text-sm.grid.grid-cols-1(v-if="playerId")
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
    teamId: {
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      // playerId: localStorage.getItem('playerId') ?? '',
      // seasonTournamentIds: [],
      // seasonTournamentYears: [],
      // tournamentsIds: [],
      idsArray: [],
      playerStatsArr: [ [], [] ],
      playerStatsArr1: [],
      playerStatsArr2: [],

      unitedValues: [],
      unitedValues1: {},
      unitedValues2: {},
      crucialMoment: [9576372],
      isLoaded: false,
      statNames: {
        minutesPlayed: 'Минут сыграно',

        // touches: 'Касаний (действий с мячом) на 90\'',
        // totalPass: 'Пасов на 90\'',
        // accuratePass: 'Успешных передач на 90\'',

        // bigChanceCreated: 'Явных голевых моментов создано на 90\'',
        assists: 'Ассистов на 90\'',

        // wonContest: 'Успешного дриблинга на 90\'',
        // totalContest: 'Попыток дриблинга на 90\'',

        // wasFouled: 'Заработано фолов',

        // duelWon: 'Выигранных наземных дуэлей на 90\'',
        // duelLost: 'Проигранных наземных дуэлей',


        // dispossessed: 'Лишен мяча',
        // possessionLostCtrl: 'Потерей мяча на 90\'',

        // totalShots: 'Ударов на 90\'',
        // onTargetScoringAttempt: 'Ударов по воротам на 90\'',
        // shotOffTarget: 'Ударов мимо ворот на 90\'',
        // blockedScoringAttempt: 'Заблокированных на 90\'',

        bigChanceMissed: 'Упущено голевых моментов на 90\'',
        goals: 'Всего голов на 90\'',


        // totalCross: 'Сделано навесов на 90\'',
        // accurateCross: 'Успешных навесов на 90\'',


        // totalLongBalls: 'Длинных передачи на 90\'',
        // accurateLongBalls: 'Успешных длинных передач',
        // accurateChippedPasses: 'Успешных пасов с подсечкой',
        // keyPass: 'Ключевых передач на 90\'',

        // aerialWon: 'Выигранных воздушных единоборств на 90\'',
        // aerialLost: 'Проигранных воздушных единоборств',


        // challengeLost: 'Обыгран на дриблинге',
        // fouls: 'Совершенных фолов',

        interceptionWon: 'Перехватов на 90\'',
        totalTackle: 'Отборов на 90\'',
        totalClearance: 'Выносов на 90\'',
        outfielderBlock: 'Заблокировано ударов на 90\'',
        totalOffside: 'Офсайдов',

        // goalConversionPercentage: 'Конвертация ударов в голы в %',
        // cleanSheet: 'Сухарей',
        rating: 'Средний рейтинг sofascore',
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
  // watch: {
  //   async seasonTournamentIds() {
  //     for (const [idx, seasonId] of this.seasonTournamentIds.entries()) {
  //       console.log(seasonId)
  //       const response = await this.$axios.get(
  //         `/api/v1/player/${this.playerId}/unique-tournament/${this.tournamentsIds[idx]}/season/${this.seasonTournamentIds[idx]}/statistics/overall`
  //       )
  //       const statsResponce = response.data.statistics
  //       // данные для длины блоков
  //       // {app: [], touches: [] ...}
  //       for (const statName in this.statNames) {
  //         if (!Array.isArray(this.tableNumbers[statName]))
  //           this.tableNumbers[statName] = []
  //         this.tableNumbers[statName].push(statsResponce[statName])
  //       }
  //     }
  //     console.log('this.isLoaded')

  //     this.isLoaded = true
  //     this.getStatByMatch()
  //     console.log(this.isLoaded)
  //   },
  // },
  methods: {
    async fetchMatchesIds() {
      for (let i = 0; i < 2; i++) {
        try {
          const response = await this.$axios.get(
            `/api/v1/team/${this.teamId}/events/last/${i}`
          )
          const eventArray = response.data.events
          const ids = eventArray.map((match) => match.id)
          // найти способ соединять массивы в цикле
          this.idsArray.push(ids)
        } catch (e) {
          console.error(e)
        }
      }
      this.idsArray = [...this.idsArray[1], ...this.idsArray[0]]

      await this.fetchStatictics()
    },
    async fetchStatictics() {
      // разделяем массив на ДО и ПОСЛЕ
      const crucialMomentIndex = this.idsArray.findIndex(
        (idx) => idx === this.crucialMoment[0]
      )
      console.log(crucialMomentIndex)
      const dividedIdsArray = []

      // const firstIdsArr = this.idsArray.slice(0, crucialMomentIndex)
      // const secondIdsArr = this.idsArray.slice(crucialMomentIndex)

      for (let i = 0; i < this.crucialMoment.length + 1; i++) {
        console.log(dividedIdsArray)
        console.log(i)
        i === 0
        ? dividedIdsArray[i] = this.idsArray.slice(0, crucialMomentIndex)
        : dividedIdsArray[i] = this.idsArray.slice(crucialMomentIndex)
      }
      console.log('dividedIdsArray')
      console.log(dividedIdsArray)
      // console.log('firstIdsArr')
      // console.log(firstIdsArr)
      // console.log('secondIdsArr')
      // console.log(secondIdsArr)

      // проходимся по каждому и собираем статистику
      for (let i = 0; i < dividedIdsArray.length; i++) {
        const idsArr = dividedIdsArray[i];
        for (const matchId of idsArr) {
          try {
            const response = await this.$axios.get(
              `https://api.sofascore.com/api/v1/event/${matchId}/player/${this.playerId}/statistics`
            )
            const statInMatch = response.data.statistics
            // console.log(statInMatch)
            this.playerStatsArr[i].push(statInMatch)
            // console.log('this.playerStatsArr' + i)
            // console.log(this.playerStatsArr)
          } catch (e) {
            console.error(e)
          }
        }
      }
      console.log(this.playerStatsArr)

      // for (const matchId of secondIdsArr) {
      //   try {
      //     const response = await this.$axios.get(
      //       `https://api.sofascore.com/api/v1/event/${matchId}/player/${this.playerId}/statistics`
      //     )
      //     const statInMatch = response.data.statistics
      //     this.playerStatsArr2.push(statInMatch)
      //   } catch (e) {
      //     console.error(e)
      //   }
      // }
      this.uniteValues()
    },

    uniteValues() {
      for (let i = 0; i < this.playerStatsArr.length; i++) {
        this.unitedValues[i] = {}
        for (const statName in this.statNames) {
          console.log(statName)
          if (Object.hasOwnProperty.call(this.statNames, statName)) {
            const summValue = this.playerStatsArr[i].reduce((summ, matchStat) => {

              if (!matchStat[statName]) {
                matchStat[statName] = 0
              }

              return summ + matchStat[statName]
            }, 0)
            console.log('this.unitedValues[i].minutesPlayed')
            // console.log(summValue)
            console.log(this.unitedValues[i])

            if (statName === 'rating') this.unitedValues[i][statName] = +(summValue / this.playerStatsArr[i].length).toFixed(1)
            else if (statName === 'minutesPlayed') this.unitedValues[i][statName] = summValue
            else this.unitedValues[i][statName] = +(summValue / this.unitedValues[i].minutesPlayed * 90).toFixed(2)
            console.log(this.unitedValues[i][statName])
          }
        }

        this.unitedValues[i].matches = this.playerStatsArr[i].length
        this.unitedValues[i].minutesPlayed = this.unitedValues[i].minutesPlayed / this.unitedValues[i].matches
      }
    },


    // async fetchSeasonsTournament() {
    //   // по id игрока получаем id сезона в конкретной лиге. Зачастую нас будет интересовать нац лига, т.е. первая в списке
    //   // (возможно, придется добавлять условие для того, какую лигу я ожидаю увидеть, и вставлять название лиги в проверку, передавая через параметры)
    //   localStorage.setItem('playerId', this.playerId)
    //   try {
    //     const response = await this.$axios.get(
    //       `https://api.sofascore.com/api/v1/player/${this.playerId}/statistics/seasons`
    //     )
    //     let succesSeasons = 0
    //     while (succesSeasons < this.needSeasons.length) {
    //       for (
    //         let i = 0;
    //         i < response.data.uniqueTournamentSeasons.length;
    //         i++
    //       ) {
    //         const tournament = response.data.uniqueTournamentSeasons[i]
    //         const tournamentName = tournament.uniqueTournament.name
    //         if (
    //           tournamentName === 'Premier League' ||
    //           tournamentName === 'LaLiga' ||
    //           tournamentName === 'Ligue 1' ||
    //           tournamentName === 'Serie A' ||
    //           tournamentName === 'Bundesliga' ||
    //           tournamentName === 'Eredivisie'
    //         ) {
    //           // проверяем, есть ли в данном турнире нужный год (напр. Лукаку выступал в АПЛ в сезоне 21/22 - хотим выводить именно этот сезон в первую очередь)
    //           const hasNeedSeason = tournament.seasons.some(
    //             (season) => season.year === this.needSeasons[succesSeasons]
    //           )
    //           if (!hasNeedSeason) continue
    //           const needSeason = tournament.seasons.find(
    //             (season) => season.year === this.needSeasons[succesSeasons]
    //           )
    //           this.tournamentsIds.push(tournament.uniqueTournament.id)
    //           this.seasonTournamentIds.push(needSeason.id)
    //           this.seasonTournamentYears.push(needSeason.year)
    //           succesSeasons++
    //           if (succesSeasons === this.needSeasons.length) {
    //             i = response.data.uniqueTournamentSeasons.length
    //             break
    //           }
    //         }
    //       }
    //     }
    //   } catch (e) {
    //     console.error(e)
    //   }
    // },

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
