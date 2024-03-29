<template>
  <div>
    <br />
    <h1 style="margin-top: 62px; text-align: center">{{ $t('games.list') }}</h1>
    <h3 style="text-align: center">{{ $t('games.subtitle') }}</h3>
    <v-container>
      <v-card>
        <v-card-title>{{ $t('games.search') }}
      <v-spacer></v-spacer>
          <v-text-field
            v-model="search"
            append-icon="mdi-magnify"
            :label="$t('games.inputhere')"
            single-line
            hide-details
          ></v-text-field>
        </v-card-title>
        <v-progress-linear
          v-if="$fetchState.pending"
          indeterminate
          style="margin:7px"
          color="primary"
        ></v-progress-linear>
        <v-data-table
          v-if="!$fetchState.pending"
          :headers="headers"
          :items="games.storeContent"
          :search="search"
          :items-per-page="15"
          style="cursor:pointer;"
          loading-text="Loading... Please wait"
          class="elevation-1"
          :footer-props="{ 'items-per-page-options': [15, 30, 50, 100, -1], 'sortBy': 'item.info.title' }"
          @click:row="selectGameBtn"
        >
          <template v-slot:item.info.icon_url="{ item }">
            <img :src="item.info.icon_url" />
          </template>
        </v-data-table>
      </v-card>
      <v-dialog v-if="selectGame" v-model="dialogGame" width="800">
        <v-card outlined>
          <v-carousel
            v-if="selectGame.info.screenshots"
            :continuous="true"
            :show-arrows="false"
            cycle
            show-arrows-on-hover
            delimiter-icon="mdi-minus"
            height="300"
          >
            <v-carousel-item
              v-for="(item, i) in selectGame.info.screenshots"
              :key="i"
              :src="item.url"
            ></v-carousel-item>
          </v-carousel>
          <v-card-title>
            {{ selectGame.info.title }}
          </v-card-title>

          <v-card-text>
            Description :
            {{
              selectGame.info.description
                ? selectGame.info.description
                : $t('games.notspecified')
            }}
            <br />
            Version :
            {{
              selectGame.info.version ? selectGame.info.version : $t('games.notspecified')
            }}
            <br />
            {{ $t('games.author') }} :
            {{
              selectGame.info.author ? selectGame.info.author : $t('games.notspecified')
            }}
            <br />
            {{ $t('games.category') }} :
            {{
              selectGame.info.category
                ? selectGame.info.category
                : $t('games.notspecified')
            }}
            <br />
            Console :
            {{
              selectGame.info.console ? selectGame.info.console : $t('games.notspecified')
            }}
            <br />
            {{ $t('games.last_updated') }} :
            {{
              selectGame.info.last_updated ? selectGame.info.last_updated : $t('games.notspecified')
            }}
            <br />
          </v-card-text>

          <v-tabs
            v-if="downloadLinks[0]"
            v-model="tabGame"
            next-icon="mdi-arrow-right-bold-box-outline"
            prev-icon="mdi-arrow-left-bold-box-outline"
            show-arrows
          >
            <v-tab
              v-for="downloadLinkItem in downloadLinks"
              :key="downloadLinkItem.name"
            >
              {{ downloadLinkItem.name }}
            </v-tab>
          </v-tabs>

          <v-card flat>
            <qrcode-vue
              v-if="QRCodeURL"
              class="qrcode text-center"
              :value="QRCodeURL"
              :size="200"
            />
          </v-card>

          <v-footer>
            <v-spacer></v-spacer>
            <div :v-if="QRsize">{{ $t('games.size') }} : {{ QRsize }}</div>
            <v-btn icon :href="QRCodeURL.replace('http://', 'https://')">
              <v-icon>mdi-download</v-icon>
            </v-btn>
            <v-btn color="primary" text @click="dialogGame = false">
              {{ $t('close') }}
            </v-btn>
          </v-footer>
        </v-card>
      </v-dialog>
    </v-container>
  </div>
</template>

<script>
import QrcodeVue from 'qrcode.vue'

export default {
  components: {
    QrcodeVue,
  },
  asyncData() {
    return new Promise((resolve) => {
      // eslint-disable-next-line nuxt/no-timing-in-fetch-data
      setTimeout(function () {
        resolve({})
      }, 500)
    })
  },
  data() {
    return {
      search: '',
      games: null,
      headers: null,
      selectGame: null,
      dialogGame: false,
      downloadLinks: [],
      tabGame: null,
      QRCodeURL: undefined,
      QRsize: undefined,
    }
  },
  async fetch() {
    const gamesResponse = await this.$axios.$get(
      `https://ghosteshop.com/dbapi.php`
    )
    this.games = gamesResponse
    this.headers = [
      {
        text: this.$t('games.icon'),
        align: 'start',
        value: 'info.icon_url',
        sortable: false
      },
      {
        text: this.$t('games.title'),
        value: 'info.title',
      },
      { text: this.$t('games.description'), value: 'info.description', sortable: false },
      { text: this.$t('games.version'), value: 'info.version', sortable: false },
      { text: this.$t('games.author'), value: 'info.author' },
      { text: this.$t('games.category'), value: 'info.category' },
      { text: this.$t('games.console'), value: 'info.console' },
    ]
  },
  watch: {
    tabGame(index, item) {
      if (this.downloadLinks[index]) {
        this.QRCodeURL = this.downloadLinks[index].downloadLink
        this.QRsize = this.downloadLinks[index].downloadSize
      } else {
        this.QRCodeURL = this.downloadLinks[0].downloadLink
        this.QRsize = this.downloadLinks[0].downloadSize
      }
    },
  },
  methods: {
    selectGameBtn(item) {
      this.downloadLinks = []
      this.QRsize = undefined
      this.QRCodeURL = undefined
      for (const i in item) {
        if (i !== 'info') {
          this.downloadLinks.push({
            name: i,
            downloadLink: item[i].script[0].file,
            downloadSize: item[i].size,
          })
        }
      }
      this.QRCodeURL = this.downloadLinks[0].downloadLink
      this.QRsize = this.downloadLinks[0].downloadSize
      this.selectGame = item
      this.dialogGame = true
    },
  },
}
</script>

<style scopped>
.qrcode canvas {
  border: 15px solid #ffffff;
}

.v-window-item .v-image__image.v-image__image--cover {
  background-size: contain;
}
</style>
