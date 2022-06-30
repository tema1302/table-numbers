<template lang="pug">
  .component
    button.relative.button.inline-block.px-5.bg-blue-600.rounded-xl.text-white(@click="fetchMatchesIds") Get data
    .stats-table.text-left.text-sm.grid.grid-cols-1(v-if="playerId")
      h4.text-gray-800.text-3xl.font-black.col-span-1.uppercase.text-center Игроки "МЮ" при тренерах в сезоне 21/22
      .comparison-title.flex.justify-center.items-center
        .comparison-item.comparison-item-1
          span При Сульшере
        .comparison-image
          img(src="https://api.sofascore.app/api/v1/player/750/image")
        .comparison-item.comparison-item-2
          span При Рангнике
      .flex.flex-wrap.items-center.justify-center.mb-3(v-if="isLoaded", v-for="(russStatName, statName, idx) in statNames" :key="statName")
        .descr-title.w-full {{ russStatName[0] }} / {{ statName }}
        .cols-comparison.text-right.flex-1.flex.flex-row-reverse
          .comparison-item.comparison-item-0.text-white.text-xl(:style="{ width: overallWidth[0][statName] + '%' }") {{ unitedValues[0][statName] }}
        .flex.flex-col.items-center
          .cols-title.smile {{ russStatName[1] }}
        .cols-comparison.text-left.flex-1
          .comparison-item.comparison-item-1.text-white.text-xl(:style="{ width: overallWidth[1][statName] + '%' }") {{ unitedValues[1][statName] }}
      //- .cols-comparison(v-if="isLoaded", v-for="(russStatName, statName, idx) in statNames" :key="statName")
      //-   .stat-legend.mx-1.relative
      //-     .stat-legend-text.absolute {{ russStatName }} / {{ statName }}
      //-   .stat-value.mx-1.py-2(v-for="(value, i) in overallStatWithCombinedIndicators[statName]" :key="value")
      //-     span.range {{ statName.includes('Percentage') ? `${value} %` : value }}
      //-       span.range-width(:style="{ width: overallWidth[statName][i] + '%' }")
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
      playerStatsArr: [[], []],
      playerStatsArr1: [],
      playerStatsArr2: [],

      unitedValues: [],
      unitedValues1: {},
      unitedValues2: {},
      crucialMoment: [9576372],
      isLoaded: false,
      statNames: {
        minutesPlayed: ['Минут сыграно', '🕒'],

        touches: ["Касаний (действий с мячом) на 90'", '👣'],
        totalPass: ["Пасов на 90'", '🔗'],
        accuratePass: ["Успешных передач на 90'", '🌟'],

        bigChanceCreated: ["Явных голевых моментов создано на 90'", '🤩'],
        goalAssist: ["Ассистов на 90'", '👟'],

        wonContest: ["Успешного дриблинга на 90'", '🐰'],
        totalContest: ["Попыток дриблинга на 90'", '🏃‍♂️'],

        wasFouled: ['Заработано фолов', '😖'],

        duelWon: ["Выигранных наземных дуэлей на 90'", '🌵'],
        duelLost: ["Проигранных наземных дуэлей на 90'", '🌳'],

        dispossessed: ['Лишен мяча', '🦦'],
        possessionLostCtrl: ["Потерей мяча на 90'", '🦥'],
        // ☄️
        onTargetScoringAttempt: ["Ударов в створ на 90'", '🎯'],
        shotOffTarget: ["Ударов мимо ворот на 90'", '😵'],
        blockedScoringAttempt: ["Заблокированных на 90'", '🧱'],

        bigChanceMissed: ["Упущено голевых моментов на 90'", '🥅❌'],
        goals: ["Всего голов на 90'", '⚽'],

        totalCross: ["Сделано навесов на 90'", '🤼'],
        accurateCross: ["Успешных навесов на 90'", '👌'],

        totalLongBalls: ["Длинных передачи на 90'", '↗'],
        accurateLongBalls: ['Успешных длинных передач', '👍'],
        // accurateChippedPasses: ['Успешных пасов с подсечкой', '🔗'],
        keyPass: ["Ключевых передач на 90'", '🔑'],

        aerialWon: ["Выигранных воздушных единоборств на 90'", '🦒'],
        aerialLost: ['Проигранных воздушных единоборств на 90', '🌱'],

        challengeLost: ['Обыгран на дриблинге', '✈️'],
        fouls: ['Совершенных фолов', '👊'],

        interceptionWon: ["Перехватов на 90'", '🔥'],
        totalTackle: ["Отборов на 90'", '🦏'],
        totalClearance: ["Выносов на 90'", '👑'],
        outfielderBlock: ["Заблокировано ударов на 90'", '😈'],
        totalOffside: ['Офсайдов', '🧠'],

        // goalConversionPercentage: ['Конвертация ударов в голы в %', ''],
        // cleanSheet: ['Сухарей', ''],
        rating: ['Средний рейтинг sofascore', '🚀'],
      },
      tableNumbers: {},
      statsForBlocksWidth: {},
      overallStatByMatch: {},

      overallWidth: [],
    }
  },

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
          ? (dividedIdsArray[i] = this.idsArray.slice(0, crucialMomentIndex))
          : (dividedIdsArray[i] = this.idsArray.slice(crucialMomentIndex))
      }
      console.log('dividedIdsArray')
      console.log(dividedIdsArray)

      // проходимся по каждому и собираем статистику
      for (let i = 0; i < dividedIdsArray.length; i++) {
        const idsArr = dividedIdsArray[i]
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
      this.uniteValues()
    },

    uniteValues() {
      for (let i = 0; i < this.playerStatsArr.length; i++) {
        this.unitedValues[i] = {}
        for (const statName in this.statNames) {
          // проверка на то, есть ли свойство с таким названием statName в объекте
          if (Object.hasOwnProperty.call(this.statNames, statName)) {
            // вычисляю сумму отдельно взятого свойства (напр., ассистов)
            const summValue = this.playerStatsArr[i].reduce(
              (summ, matchStat) => {
                if (!matchStat[statName]) {
                  matchStat[statName] = 0
                }

                return summ + matchStat[statName] // данную сумму всех свойств используем для расчетов
              },
              0
            )

            // если рейтинг, то просто сумму всех оценок делим на кол-во матчей
            if (statName === 'rating')
              this.unitedValues[i][statName] = +(
                summValue / this.playerStatsArr[i].length
              ).toFixed(2)
            // если количество минут, то банально присваиваем значение
            else if (statName === 'minutesPlayed')
              this.unitedValues[i][statName] = +summValue
            // если любая другая стата, то узнаем количество на 90 минут
            else
              this.unitedValues[i][statName] = +(
                (summValue / this.unitedValues[i].minutesPlayed) *
                90
              ).toFixed(2)
            console.log(this.unitedValues[i][statName])
          }
        }

        this.unitedValues[i].matches = this.playerStatsArr[i].length
        this.unitedValues[i].minutesPlayed = +(
          this.unitedValues[i].minutesPlayed / this.unitedValues[i].matches
        ).toFixed(1)
      }
      console.log('this.unitedValues[0]')
      console.log(this.unitedValues[0])
      console.log('this.unitedValues[1]')
      console.log(this.unitedValues[1])

      this.setOverallWidth()
    },

    setOverallWidth() {
      for (let k = 0; k < this.unitedValues.length; k++) {
        this.overallWidth[k] = {}
        for (const statName in this.unitedValues[0]) {
          const comparisonArray = []
          for (let i = 0; i < this.unitedValues.length; i++) {
            const statValue = this.unitedValues[i][statName]
            comparisonArray.push(statValue)
          }
          const maxValue = Math.max(...comparisonArray)
          if (maxValue === 0) this.overallWidth[k][statName] = 0
          else {
            const statValuesPersentage = comparisonArray.map((value) =>
              Math.round((value / maxValue) * 100)
            )
            this.overallWidth[k][statName] = statValuesPersentage[k]
          }
          console.log('overallWidth' + k)
          console.log(this.overallWidth[k])
        }
      }
      this.isLoaded = true
      this.$forceUpdate()

      // const overallArr = Object.keys(this.overallStatWithCombinedIndicators)
      // for (const statName of overallArr) {
      //   let statValues = this.overallStatWithCombinedIndicators[statName]
      //   if (typeof statValues[0] === 'string') {
      //     statValues = statValues.map((val) => {
      //       console.log(val)
      //       if (val.slice(-7, -3) === '00.0') return +val.slice(-8, -3)
      //       console.log(+val.slice(-7, -3))
      //       return +val.slice(-7, -3)
      //     })
      //   }
      //   console.log(statValues)
      //   const maxValue = Math.max(...statValues)
      //   const statValuesPersentage = statValues.map((value) => {
      //     return Math.round((value / maxValue) * 100)
      //   })
      //   this.overallWidth[statName] = statValuesPersentage
      // }
    },
  },
}
</script>

<style lang="scss">
$blueGradient: linear-gradient(
  to right top,
  #5758b7,
  #525cb7,
  #4e60b7,
  #4a64b6,
  #4767b5,
  #446fbb,
  #4277c1,
  #407fc6,
  #3d8ed3,
  #3b9edf,
  #3badea,
  #40bdf4
);
$redGradient: linear-gradient(
  to right top,
  #d41c95,
  #df067c,
  #e40763,
  #e31d4a,
  #de3232
);

.component:first-child .cols-comparison .stat-legend {
  opacity: 1;
}
.component .cols-comparison .stat-legend {
  opacity: 0;
}
.stats-table {
  padding: 15px;
  border: 1px solid rgba($color: #0a0a0a, $alpha: 0.1);
  margin: 32px 0 0;
  color: #221d1d;
  position: relative;
  z-index: 1;
  // скрываем матчи (хотя один раз нужно будет показать)
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

  .comparison-title {
    transform: translateY(-10px);
  }

  @mixin comparison-1 {
    border-radius: 24px 0 0 24px;
    text-align: right;
    background-image: $blueGradient;
  }
  @mixin comparison-2 {
    border-radius: 0 24px 24px 0;
    text-align: left;
    background-image: $redGradient;
  }

  .comparison-title {
    .comparison-image {
      max-width: 132px;
      position: relative;
      z-index: 2;
      img {
        border-radius: 50%;
      }
    }
    .comparison-item {
      width: 220px;
      padding: 12px 28px;
      font-size: 20px;
      color: #ffffff;
      position: relative;
      z-index: 1;
    }
    .comparison-item-1 {
      @include comparison-1;
      transform: translateX(16px);
    }
    .comparison-item-2 {
      @include comparison-2;
      transform: translateX(-16px);
    }
  }

  .cols-title {
    width: 60px;
    border-radius: 12px;
    height: 60px;
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    border: 1px solid rgba($color: #0a0a0a, $alpha: 0.1);
    font-size: 36px;
  }

  .descr-title {
    flex: 0 0 100%;
    text-align: center;
  }

  .comparison-item {
    padding: 0px 14px;
  }
  .comparison-item-0 {
    @include comparison-1;
    direction: rtl;
  }
  .comparison-item-1 {
    @include comparison-2;
  }
  .stat-legend {
    min-height: 50px;
    font-size: 13px;
    color: #221d1d;
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
