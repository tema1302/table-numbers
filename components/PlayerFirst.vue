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
      .cols-comparison(v-if="isLoaded", v-for="(value, statName) in statNames" :key="value")
        .stat-legend.mx-1.py-3 {{ value }}
        .stat-value.mx-1.py-2(v-for="(value, i) in overallStatByMatch[statName]" :key="i")
          span.range {{ statName.includes('Percentage') ? `${value} %` : value }}
            span.range-width(v-if="Object.keys(overallWidth).length > 0", :style="{ width: overallWidth[statName][i] + '%' }")
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
      needSeasons: ['21/22', '20/21', '19/20', '18/19'],
      statNames: {
        appearances: 'Матчей в сезоне',

        // matchesStarted: 'Из них в старте',
        // touches: 'Касаний за игру',
        // totalPasses: 'Пасов за игру',

        // totalOppositionHalfPasses: 'Пасов на чужую половину поля',
        // accuratePassesPercentage: '% успешных пасов',

        // successfulDribbles: 'Успешного дриблинга за игру',
        // successfulDribblesPercentage: '% успешного дриблинга',

        // bigChancesCreated: 'Явных голевых моментов создано',
        // bigChancesMissed: 'Упущено голевых моментов',
        // assists: 'Ассистов',
        // possessionWonAttThird: 'Возвратов владения (финальная треть)',

        // totalShots: 'Ударов за игру',

        // shotsFromOutsideTheBox: 'Ударов за пределами штрафной за игру',
        // shotsOnTarget: 'Ударов по воротам за игру',
        // shotsOffTarget: 'Ударов мимо ворот за игру',

        // goals: 'Всего голов',
        // goalsAssistsSum: 'Гол + пас',
        // keyPasses: 'Ключевых передач за игру',

        // penaltyGoals: 'Голов из пенальти',
        // goalConversionPercentage: 'Конвертация ударов в голы в %',


        // totalLongBalls: 'Длинных передачи за игру',
        // accurateLongBallsPercentage: '% успешных длинных передач',
        // accurateChippedPasses: 'Успешных пасов с подсечкой',

        // accurateFinalThirdPasses: 'Успешных передач в финальную треть',

        // aerialDuelsWon: 'Выигранных воздушных единоборств за игру',
        // aerialDuelsWonPercentage: '% выигранных воздушных единоборств',

        // groundDuelsWon: 'Выигранных наземных дуэлей за игру',
        // groundDuelsWonPercentage: '% выигранных наземных дуэлей',


        // на всякий случай
        possessionLost: 'Потерей мяча за игру',
        interceptions: 'Перехватов за игру',
        tackles: 'Отборов за игру',
        clearances: 'Выносов за игру',

        // blockedShots: 'Заблокировано ударов за игру',
        // cleanSheet: 'Сухарей',
      },
      tableNumbers: {},
      statsForBlocksWidth: {},
      overallStatByMatch: {},
    }
  },

  computed: {
    overallWidth() {
      const rtn = {}
      const overallArr = Object.keys(this.statNames)
      for (const statName of overallArr) {
        const statValues = this.overallStatByMatch[statName]
        // для бичей, у которых точность паса низкая
        // if (statName === 'accuratePassesPercentage') {
        //   rtn[statName] = statValues
        //   continue
        // }
        const maxValue = Math.max(...statValues)
        const statValuesPersentage = statValues.map((value) => {
          return Math.round((value / maxValue) * 100)
        })
        rtn[statName] = statValuesPersentage
      }
      return rtn
    },
  },
  watch: {
    async seasonTournamentIds() {
      for (const [idx, seasonId] of this.seasonTournamentIds.entries()) {
        console.log(idx)
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
      this.getOverallStatByMatch()
      this.isLoaded = true
      // console.log(this.overallStatByMatch)
    },
  },
  methods: {
    async fetchSeasonsTournament() {
      // по id игрока получаем id сезона в конкретной лиге. Зачастую нас будет интересовать нац лига, т.е. первая в списке
      // (возможно, придется добавлять условие для того, какую лигу я ожидаю увидеть, и вставлять название лиги в проверку, передавая через параметры)
      localStorage.setItem('playerId', this.playerId)
      try {
        const response = await this.$axios.get(
          `https://api.sofascore.com/api/v1/player/${this.playerId}/statistics/seasons`
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
    getOverallStatByMatch() {
      const rtn = {}
      for (const statName in this.tableNumbers) {
        if (Object.hasOwnProperty.call(this.tableNumbers, statName)) {
          const arrStatValue = this.tableNumbers[statName]
          if (
            statName === 'matchesStarted' ||
            statName === 'appearances' ||
            statName === 'cleanSheet' ||
            statName === 'assists' ||
            statName === 'bigChancesCreated' ||
            statName === 'penaltyGoals' ||
            statName === 'goals' ||
            statName === 'bigChancesMissed' ||
            statName === 'goalsAssistsSum'
          ) {
            rtn[statName] = arrStatValue
          } else {
            const arrStatValueByMatch = arrStatValue.map((statValue, idx) => {
              if (statName.includes('Percentage')) {
                return +statValue.toFixed(1)
              } else {
                return +(
                  statValue / this.tableNumbers.appearances[idx]
                ).toFixed(2)
              }
            })
            // console.log(statName)
            // console.log(arrStatValueByMatch)
            rtn[statName] = arrStatValueByMatch
          }
        }
      }
      this.overallStatByMatch = rtn
    },
  },
}
</script>

<style lang="scss">
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
  h4 {
    min-height: 65px;
  }
  .stat-legend {
    min-height: 50px;
    font-size: 13px;
    color: rgba($color: #fff, $alpha: 0.85);
    display: flex;
    align-items: flex-end;
    line-height: 1;
  }
  .stat-value {
    position: relative;
    text-align: left;
    padding-left: 5px;
    font-size: 16px;
    font-family: 'Open Sans Condensed', sans-serif;
    .range {
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
  .matches,
  .touches {
    // max-width: 62px;
  }
}
</style>
