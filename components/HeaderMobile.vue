<template>
  <v-app-bar v-if="$vuetify.breakpoint.mobile" app color="white">
    <v-app-bar-nav-icon @click.stop="$store.commit('setDrawerMobile', true)" />
    <v-img src="/favicon.ico" max-width="32px" class="mr-1 mb-1" />
    <v-toolbar-title class="font-weight-black">Ghost eShop</v-toolbar-title>
  </v-app-bar>
</template>

<script>
export default {
  data() {
    return {
      listLang: null,
      lang: [
        {
          name: this.$t('header.english'),
          lang: 'en',
        },
        {
          name: this.$t('header.french'),
          lang: 'fr',
        },
        {
          name: this.$t('header.spanish'),
          lang: 'es',
        },
        {
          name: this.$t('header.japanese'),
          lang: 'jp',
        },
      ],
    }
  },
}
</script>

<style scopped>
.v-btn {
  margin-left: 10px;
}

.noActive::before {
  opacity: 0 !important;
}
</style>