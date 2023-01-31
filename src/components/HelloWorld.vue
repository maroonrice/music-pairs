<template>
  <v-container>
    <v-row class="text-center" v-if="initialize.show">
      <v-col cols="12">
        <v-card class="mx-auto">
          <v-toolbar
            flat
            color="deep-purple-accent-4"
            dark
          >
            <v-toolbar-title>Game Initialize</v-toolbar-title>
          </v-toolbar>
          <v-card-text>
            <h2 class="text-h6 mb-2">
              Target Musics
            </h2>

            <v-chip-group
              v-model="initialize.target_music"
              column
              multiple
            >
              <v-chip v-for="music in musics"
                filter
                variant="outlined"
              >
              {{ music.name }}
              </v-chip>
            </v-chip-group>

            <h2 class="text-h6 mb-2">
              Players
            </h2>
            <v-slider
              v-model="initialize.player"
              :min="1"
              :max="10"
              :step="1"
              thumb-label
            ></v-slider>

            <v-btn @click="start">Start!!</v-btn>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
    <v-row class="text-center" v-if="!initialize.show">
      <v-col cols="12">
        <v-card class="mx-auto mb-2" v-show="!game.result">
          <v-toolbar
            flat
            color="deep-purple-accent-4"
            dark
          >
            <v-toolbar-title>Music Pairs</v-toolbar-title>
          </v-toolbar>
          <v-card-text>

            <div class="d-flex justify-space-between mb-2">
              <v-btn @click="stop">Music Stop</v-btn>
              <v-btn @click="end">Game End</v-btn>
            </div>

            <h2 class="text-h6 mb-2">
              Player
            </h2>
            <v-radio-group inline v-model="game.current">
              <v-radio v-for="player in game.player" :label="player.name" :value="player.id"></v-radio>
            </v-radio-group>

            <h2 class="text-h6 mb-2">
              Select Music Pair
            </h2>
            <v-text-field
              :label="'First Music Number ( 1 - ' + game.musics.length + ' )'"
              type="number"
              min="1"
              :max="game.musics.length"
              v-model="game.first.choose"
            ></v-text-field>
            <div class="d-flex justify-space-between mb-2">
              <v-btn @click="firstDecide" block :disabled="game.first.decide !== undefined">Decide</v-btn>
            </div>
            <iframe v-if="this.game.first.preview" v-show="this.game.first.show" width="100%" height="100%" :src="game.first.preview" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" class="mb-2"></iframe>

            <v-text-field
              :label="'Second Music Number ( 1 - ' + game.musics.length + ' )'"
              type="number"
              min="1"
              :max="game.musics.length"
              v-model="game.second.choose"
            ></v-text-field>
            <div class="d-flex justify-space-between mb-2">
              <v-btn @click="secondDecide" block :disabled="game.first.decide === undefined || game.second.decide !== undefined">Decide</v-btn>
            </div>
            <iframe v-if="this.game.second.preview" v-show="this.game.second.show" width="100%" height="100%" :src="game.second.preview" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" class="mb-2"></iframe>

          </v-card-text>
        </v-card>
        <v-expansion-panels class="mb-2" v-for="player in game.player">
          <v-expansion-panel :title="player.name">
            <v-expansion-panel-text v-for="music in player.musics">
              {{ music.name }}
            </v-expansion-panel-text>
          </v-expansion-panel>
        </v-expansion-panels>

        <div class="d-flex justify-space-between mb-2">
          <v-btn @click="stop">Music Stop</v-btn>
          <v-btn @click="end">Game End</v-btn>
        </div>
      </v-col>
    </v-row>
  </v-container>
  <v-snackbar
    v-model="snackbar.show"
  >
    {{ snackbar.message }}

    <template v-slot:actions>
      <v-btn
        color="pink"
        variant="text"
        @click="snackbar.show = false"
      >
        Close
      </v-btn>
    </template>
  </v-snackbar>
</template>

<script>
function initGame() {
  return {
      musics: [],
      opened: [],
      player: [],
      current: 0,
      first: {
        choose: 1,
        decide: undefined,
        preview: undefined,
        show: false,
      },
      second: {
        choose: 1,
        decide: undefined,
        preview: undefined,
        show: false,
      },
      result: false
    }
}

export default {
  name: 'Music Pairs',

  data: () => ({
    musics: [
      { name: "未来予報ハレルヤ！", url: "https://www.youtube.com/embed/BHzYNQ-irmU?start=61&autoplay=1" },
      { name: "未来は風のように", url: "https://www.youtube.com/embed/A1bleJVhZeY?start=27&autoplay=1" },
      { name: "START!! True dreams", url: "https://www.youtube.com/embed/HGkoec4genw?autoplay=1" },
      { name: "Tiny Stars", url: "https://www.youtube.com/embed/gmpCJnMfMR4?autoplay=1" },
      { name: "常夏☆サンシャイン", url: "https://www.youtube.com/embed/XR3h_agJgXQ?start=5&autoplay=1" },
      { name: "Wish Song", url: "https://www.youtube.com/embed/KL2l7nH8Uus?autoplay=1" },
      { name: "ノンフィクション!!", url: "https://www.youtube.com/embed/_oZ3EcOO5xU?start=2&autoplay=1" },
      { name: "Starlight Prologue", url: "https://www.youtube.com/embed/Ccyjkq75T9E?start=20&autoplay=1" },
      { name: "WE WILL!!", url: "https://www.youtube.com/embed/2tJGGmJhm_E?autoplay=1" },
      { name: "Welcome to 僕らのセカイ", url: "https://www.youtube.com/embed/eF-mzOHunEc?autoplay=1" },
      { name: "追いかける夢の先で", url: "https://www.youtube.com/embed/1DK_oVU9X1M?start=7&autoplay=1" },
      { name: "Butterfly Wing", url: "https://www.youtube.com/embed/4qmLnVCXQgY?start=5&autoplay=1" },
      { name: "Go!! リスタート", url: "https://www.youtube.com/embed/uiz8kU2JKZM?autoplay=1" },
      { name: "ビタミンSUMMER！", url: "https://www.youtube.com/embed/_rLihz4cQ4E?autoplay=1" },
      { name: "Chance Day, Chance Way！", url: "https://www.youtube.com/embed/4wKBNVE_DVU?autoplay=1" },
      { name: "Sing！Shine！Smile！", url: "https://www.youtube.com/embed/EY4_wcu7iI0?autoplay=1" },
      { name: "エーデルシュタイン", url: "https://www.youtube.com/embed/aVbKE7cZxck?start=2&autoplay=1" },
      { name: "未来の音が聴こえる", url: "https://www.youtube.com/embed/PjvpC-o1Sjk?start=6&autoplay=1" },
    ],
    initialize: {
      target_music: [],
      player: 2,
      show: true
    },
    game: initGame(),
    snackbar: {
      show: false,
      message: ""
    }
  }),
  methods: {
    random(max) {
      return Math.floor(Math.random() * max);
    },
    start() {
      this.initialize.show = false
      for (
        let targets = this.initialize.target_music.concat(this.initialize.target_music);
        targets.length > 0;
        ) {
          const num = targets.splice(this.random(targets.length),1)
          this.game.musics.push(num[0])
      }
      console.log(this.game.musics);
      (Array.from(new Array(this.initialize.player))).forEach((_,i) => {
        this.game.player.push({id: i, name: "player " + (i+1), musics:[]})
      })
    },
    firstDecide() {
      this.game.first.decide = this.game.first.choose - 1
      this.game.first.preview = this.first_music.url
      this.game.second.preview = undefined
    },
    secondDecide() {
      this.game.second.decide = this.game.second.choose - 1
      this.game.first.preview = undefined
      this.game.second.preview = this.second_music.url
      if (this.game.first.decide == this.game.second.decide) {
        this.addSnackbar(this.game.player[this.game.current].name + " selects Same Music Number")
      }
      else if (this.game.opened.includes(this.game.first.decide)
        || this.game.opened.includes(this.game.second.decide)
      ) {
        this.addSnackbar(this.game.player[this.game.current].name + " selects Opened Music Number")
      }
      else {
        if (this.first_music.name == this.second_music.name) {
          this.game.player[this.game.current].musics.push(this.first_music)
          this.game.opened.push(this.game.first.decide)
          this.game.opened.push(this.game.second.decide)
          this.addSnackbar(this.game.player[this.game.current].name + " gets Music Pair!!")
        }
      }
      this.game.first.decide = undefined
      this.game.second.decide = undefined
      this.game.current = (this.game.current + 1) % this.game.player.length
      if (this.game.musics.length == this.game.opened.length) {
        this.game.result = true
        this.addSnackbar("Game End")
      }
    },
    stop() {
      this.game.first.preview = undefined
      this.game.second.preview = undefined
    },
    end() {
      this.initialize.show = true
      this.game = initGame()
    },
    addSnackbar(message) {
      this.snackbar.show = true
      this.snackbar.message = message
    }
  },
  mounted() {
    (Array.from(new Array(this.musics.length))).forEach((_,i) => {
      this.initialize.target_music.push(i)
    })
  },
  computed: {
    first_music() {
      return this.musics?.[this.game.musics?.[this.game.first.decide]]
    },
    second_music() {
      return this.musics?.[this.game.musics?.[this.game.second.decide]]
    },
    selectable_musics() {
      return this.game.musics.filter((v,i) => !this.game.opened.includes(i))
    }
  }
}
</script>
