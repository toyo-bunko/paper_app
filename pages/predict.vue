<template>
  <div>
    <v-container class="py-5">
      <h1 class="my-5">皮紙・竹紙の分類アプリ</h1>
      <p>
        アップロードされた画像が皮紙か竹紙かを機械が判別します。500倍の画像の利用をおすすめします。
      </p>
      <template v-if="!imageUrl">
        <v-btn
          x-large
          depressed
          :loading="isSelecting"
          class="my-10"
          color="primary"
          block
          @click="onButtonClick"
          ><v-icon class="mr-2">mdi-upload</v-icon> NEW FILE</v-btn
        >
        <input
          ref="uploader"
          class="d-none"
          type="file"
          accept="image/*"
          @change="onFileChanged"
        />
      </template>
      <template v-else>
        <v-row v-if="imageUrl" class="mt-5">
          <v-col cols="12" :sm="8">
            <v-card no-body flat class="mb-4">
              <v-img :src="imageUrl" max-height="300px" contain>
                <v-row class="fill-height ma-0" align="center" justify="center">
                  <v-progress-circular
                    v-if="loading"
                    indeterminate
                    color="grey lighten-5"
                  ></v-progress-circular>
                </v-row>
              </v-img>
            </v-card>
          </v-col>
          <v-col cols="12" :sm="4">
            <v-card no-body class="mb-4">
              <div class="pa-4">
                <div v-for="(obj, key) in confidences" :key="key" class="mb-2">
                  <v-row>
                    <v-col>{{ obj.label }}</v-col>
                    <v-col class="text-right">{{ obj.value }}%</v-col>
                  </v-row>
                  <v-progress-linear :value="obj.value"></v-progress-linear>
                </div>
              </div>
            </v-card>

            <div class="text-right">
              <v-btn depressed large class="ma-1" @click="imageUrl = ''"
                ><v-icon class="mr-2">mdi-reload</v-icon> RESET</v-btn
              >
              <!-- 
              <v-btn depressed class="ma-1" color="primary"
                ><v-icon>mdi-upload</v-icon> NEW FILE</v-btn
              >
              -->
            </div>
          </v-col>
        </v-row>
      </template>

      <h2 class="my-5">{{ $t('example') }}</h2>
      <v-row>
        <v-col v-for="(obj, index) in items" :key="index" cols="12" :sm="3">
          <v-card no-body outlined class="mb-4" @click="predict(obj.image)">
            <v-img :src="obj.image"></v-img>

            <div class="pa-4">
              <h4>{{ obj.label }}</h4>
            </div>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Watch } from 'nuxt-property-decorator'
import axios from 'axios'

@Component({})
export default class about extends Vue {
  baseUrl: any = process.env.BASE_URL

  selectedFile: any = null
  isSelecting: boolean = false

  onButtonClick() {
    this.isSelecting = true
    window.addEventListener(
      'focus',
      () => {
        this.isSelecting = false
      },
      { once: true }
    )

    const uploader: any = this.$refs.uploader
    uploader.click()
  }

  async onFileChanged(e: any) {
    this.loading = true
    const file = e.target.files[0]
    this.selectedFile = file

    const reader = new FileReader()

    const self = this
    reader.onload = function (e: any) {
      self.imageUrl = e.target.result
    }

    reader.readAsDataURL(file) // convert to base64 string

    const params = new FormData()
    params.append('file', file)

    const results = await axios.post(
      'https://desolate-hollows-71015.herokuapp.com/upload',
      params
    )
    this.confidences = this.convert2confidences(results)
    this.loading = false
    // do something
  }

  loading: boolean = false

  confidences: any[] = []

  imageUrl: string = ''

  @Watch('imageUrl')
  changeImageUrl() {
    this.confidences = []
  }

  items: any[] = [
    {
      image:
        'https://raw.githubusercontent.com/nakamura196/tmp/main/PCG006%C3%97500-5.jpg', // this.baseUrl + '/data/PDG007×500-3.jpg',
      label: 'PCG006（皮紙）',
    },
    {
      image:
        'https://raw.githubusercontent.com/nakamura196/tmp/main/ZDH018%C3%97500-5.jpg', // this.baseUrl + '/data/ZDH018×500-5.jpg',
      label: 'ZDH018（竹紙）',
    },
  ]

  labels: string[] = ['皮紙', '竹紙']

  async predict(url: string) {
    // url = 'https://magazine-cf.techacademy.jp/wp-content/uploads/2020/09/08094518/68ac7aa37aacb50a5e95cf13e937aa2a.png'
    this.imageUrl = url

    this.loading = true

    const results: any = await axios.get(
      'https://desolate-hollows-71015.herokuapp.com/predict?url=' + url
    )

    this.confidences = this.convert2confidences(results)
    this.loading = false
  }

  head() {
    const title = (this as any).$t('Predict')
    return {
      title,
    }
  }

  convert2confidences(results: any) {
    const confidences = results.data.Confidences

    const obj: any = {}
    const labels = (this as any).labels
    for (let i = 0; i < labels.length; i++) {
      const label = labels[i]
      obj[label] = Math.floor(confidences[i] * 100)
    }

    const arr = Object.keys(obj).map((e) => ({ key: e, value: obj[e] }))

    arr.sort(function (a, b) {
      if (a.value < b.value) return 1
      if (a.value > b.value) return -1
      return 0
    })

    const confidences2 = []
    for (let i = 0; i < arr.length; i++) {
      const e = arr[i]
      confidences2.push({
        label: e.key,
        value: e.value,
      })
    }

    return confidences2
  }
}
</script>
