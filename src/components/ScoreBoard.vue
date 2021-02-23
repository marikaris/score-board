<template>
  <b-container fluid class="scoreboard">
    <b-row class="header text-center p-3 pt-5">
      <b-col cols="10">
        <h1 class="title">Yuseigachi Norg Quiz</h1>
      </b-col>
      <b-col cols="2">
        <img src="../assets/logo.png" alt="logo" class="logo float-right"/>
      </b-col>
    </b-row>
    <b-row>
      <b-col>
        <b-navbar toggleable="lg" type="dark" variant="dark">
          <b-collapse id="nav-collapse" is-nav>
            <b-navbar-nav>
              <b-nav-item href="#" v-b-modal.modal-1><i class="fas fa-cog"></i></b-nav-item>
              <b-modal id="modal-1" title="Score updaten">
                <b-form @submit="onSubmit" @reset="onReset">
                  <b-form-group label="Kolom afscheidingsteken">
                    <b-form-input v-model="separator"></b-form-input>
                  </b-form-group>
                  <b-form-group label="Onderwerp" v-slot="{ ariaDescribedby }">
                    <b-form-radio v-model="selected" :aria-describedby="ariaDescribedby" name="some-radios"
                                  value="judo">
                      Judo
                    </b-form-radio>
                    <b-form-radio v-model="selected" :aria-describedby="ariaDescribedby" name="some-radios"
                                  value="knowledge">
                      Algemene kennis
                    </b-form-radio>
                    <b-form-radio v-model="selected" :aria-describedby="ariaDescribedby" name="some-radios"
                                  value="music">
                      Muziek
                    </b-form-radio>
                    <b-form-radio v-model="selected" :aria-describedby="ariaDescribedby" name="some-radios" value="tv">
                      Films en TV
                    </b-form-radio>
                    <b-form-radio v-model="selected" :aria-describedby="ariaDescribedby" name="some-radios"
                                  value="sport">
                      Sport
                    </b-form-radio>
                    <b-form-radio v-model="selected" :aria-describedby="ariaDescribedby" name="some-radios"
                                  value="club">
                      Yuseigachi Norg
                    </b-form-radio>
                  </b-form-group>
                  <b-form-group>
                    <b-form-textarea
                        id="textarea"
                        v-model="text"
                        placeholder="Plak een tab gescheiden lijst van namen en hun scores"
                        rows="3"
                        max-rows="6"
                    ></b-form-textarea>
                  </b-form-group>

                  <b-button type="submit" variant="primary">Lijst updaten</b-button>
                  <b-button type="reset" variant="danger">Lijst leegmaken</b-button>
                </b-form>
              </b-modal>
              <b-nav-item href="#" @click="start"><i class="fas fa-play"></i></b-nav-item>
              <b-nav-item href="#" @click="stop"><i class="fas fa-stop"></i></b-nav-item>
            </b-navbar-nav>
          </b-collapse>
        </b-navbar>
      </b-col>
    </b-row>
    <b-row>
      <b-col>
        <b-container>
          <b-row class="tables mt-3 pt-3">
            <b-col cols="2">
              <b-table :fields="posFields" :items="generatedPlaces">
                <template #cell(icon)="data">
                  <span v-html="data.value"></span>
                </template>
              </b-table>
            </b-col>
            <b-col cols="10">
              <b-table :items="sortedScore" :fields="scoreFields"></b-table>
            </b-col>
          </b-row>
        </b-container>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
export default {
  name: 'ScoreBoard',
  data () {
    return {
      posFields: [
        {key: 'position', label: 'Positie'},
        {key: 'icon', label: ''}
      ],
      scoreFields: [
        {key: 'name', label: 'Naam'},
        {key: 'total', label: 'Totaal'},
        {key: 'judo', label: 'Judo'},
        {key: 'knowledge', label: 'Kennis'},
        {key: 'music', label: 'Muziek'},
        {key: 'tv', label: 'TV'},
        {key: 'sport', label: 'Sport'},
        {key: 'club', label: 'Yuseigachi'}
      ],
      scorePerName: {},
      nameLabel: 'Hoe heet je?',
      pointLabel: 'Total points',
      text: '',
      selected: '',
      score: [],
      separator: '\t'
    }
  },
  methods: {
    start() {
      this.$confetti.start();
    },

    stop() {
      this.$confetti.stop();
    },
    parseInput (input) {
      if (this.separator === '\\t' || this.separator === 'tab') {
        this.separator = '\t'
      } else if (this.separator !== '\t' && this.separator.startsWith('\t')) {
        this.separator.replace('\t', '')
      }
      return input.split('\n').map((row) => { return row.split(this.separator)})
    },
    onSubmit (event) {
      event.preventDefault()
      if (this.selected !== '') {
        const data = this.parseInput(this.text)
        const header = data[0]
        const namePos = header.indexOf(this.nameLabel)
        const pointPos = header.indexOf(this.pointLabel)
        data.slice(1).forEach((item) => {
          const nameLower = item[namePos].toLowerCase().replaceAll(" ", "")
          const points = item[pointPos]
          if (nameLower in this.scorePerName) {
            this.scorePerName[nameLower][this.selected] = points
          } else {
            this.scorePerName[nameLower] = {
              name: item[namePos]
            }
            this.scorePerName[nameLower][this.selected] = points
          }
        })

        let score = Object.values(this.scorePerName).map((element) => {
          return this.getTotal(element)
        })
        this.score = score
      }
    },
    onReset (event) {
      event.preventDefault()
      this.selected = ''
      this.text = ''
    },
    getTotal (element) {
      element['total'] = 0
      Object.keys(element).forEach((key) => {
        if (key !== 'name' && key !== 'total') {
          const number = parseInt(element[key])
          element['total'] += number
        }
      })
      return element
    }
  },
  computed: {
    sortedScore: function () {
      const score = this.score
      return score.sort((item1, item2) => {
        if (item1.total > item2.total) {
          return -1
        } else if (item1.total < item2.total) {
          return 1
        } else {
          return 0
        }
      })
    },
    generatedPlaces: function () {
      if (this.sortedScore.length > 0) {
        let i = 0
        let place = this.sortedScore.map(() => {
          i++
          return {position: i, icon: '<i class="far fa-smile-beam"></i>'}
        })
        place[0].icon = '<i class="fas fa-trophy"></i>'
        if (place.length > 1) {
          place[1].icon = '<i class="fas fa-award"></i>'
        }
        if (place.length > 2) {
          place[2].icon = '<i class="fas fa-award"></i>'
        }
        return place
      } else {
        return []
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.title {
  font-family: 'Bebas Neue', cursive;
  font-size: 5rem;
}

.logo {
  width: 7rem;
}

.tables {
  background: rgba(255, 255, 255, 0.7);
}

.scoreboard {
  height: 100vh;
  background: rgb(255, 255, 255);
  background: linear-gradient(180deg, rgba(255, 255, 255, 1) 0%, rgba(44, 47, 214, 1) 100%);
}
</style>
