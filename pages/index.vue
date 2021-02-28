<template>
  <div>
    <v-container class="mt-10 pb-5">
      <v-card flat>
        <div class="mb-5 pb-5 text-center">
          <img width="60px" :src="baseUrl + '/img/ogp/logo.jpg'" />
          <h1>{{ $t(siteName) }}</h1>
          <p class="py-5" v-html="siteDesc"></p>

          <v-btn color="primary" :to="localePath({ name: 'predict' })" large
            >試してみる</v-btn
          >
        </div>
      </v-card>
    </v-container>
  </div>
</template>

<script lang="ts">
import { Vue, Component } from 'nuxt-property-decorator'

@Component({})
export default class about extends Vue {
  baseUrl: string = process.env.BASE_URL || ''
  siteName: string = process.env.siteName || ''
  siteDesc: string = (process.env.siteDesc || '').split('\n').join('<br/>')

  head() {
    const title = this.$t(this.siteName)
    return {
      titleTemplate: null,
      title,
    }
  }
}
</script>
