<template>
    <div class="home">
        <section id="search">
        <v-sheet
          class="d-flex align-center"
        >
          <v-container class="text-center">
            <v-responsive class="mx-auto" width="500">
              <v-text-field
                v-model="keyword"
                append-inner-icon="mdi-magnify"
                label="키워드를 입력해주세요."
                single-line
                hide-details
                @keyup.enter="submit"
              ></v-text-field>
            </v-responsive>
          </v-container>
        </v-sheet>
      </section>

      <v-sheet class="d-flex align-center">
        <v-container class="text-center">
          <v-responsive class="mx-auto">
            <div class="search_result" v-if="!isLoaded && !isLoading">
            </div>
            <div class="search_result" v-else-if="isLoading">
              <v-progress-circular
                :size="70"
                :width="7"
                color="purple"
                indeterminate
              ></v-progress-circular>
            </div>
            <div v-else>
              <v-container class="text-right">
                <v-row>
                  <v-col
                  cols="12"
                  sm="12"
                  md="6"
                  lg="4"
                  xl="3"
                  class="ml-auto"
                  >
                    <v-checkbox
                      v-model="emotion_checkbox"
                      label="감정분류 적용"
                      color="indigo"
                      hide-details
                    ></v-checkbox>
                    <v-card
                      class="pa-2"
                      elevation="0"
                    >
                      <v-select
                        v-model="person_value"
                        :items="person_items"
                        chips
                        label="멤버"
                        multiple
                      ></v-select>
                    </v-card>
                  </v-col>
                </v-row>
              </v-container>
              <v-row
                class="fill-height"
                align="center"
                justify="center"
                dense
              >
                <v-col
                  v-for="(item, index) in filtered_items"
                  :key="index"
                  class="d-flex child-flex"
                  cols="6"
                  sm="6"
                  md="4"
                  lg="3"
                  xl="2"
                >
                  <v-hover v-slot="{ isHovering, props }">
                    <v-img
                      :src="server_static_url + item.path"
                      :lazy-src="`assets/images/mudori_lazy.png`"
                      :elevation="isHovering ? 12 : 2"
                      :class="{ 'on-hover': isHovering }"
                      v-bind="props"
                    >
                      <template v-slot:placeholder>
                        <v-row
                          class="fill-height ma-0"
                          align="center"
                          justify="center"
                        >
                          <v-progress-circular
                            indeterminate
                            color="grey-lighten-5"
                          ></v-progress-circular>
                        </v-row>
                      </template>
                      <v-overlay
                        :model-value="isHovering"
                        contained
                        scrim="#036358"
                        class="align-center justify-center"
                      >
                        <v-btn variant="flat" @click="open_overlay(index)">크게 보기</v-btn>
                      </v-overlay>
                    </v-img>
                  </v-hover>
                </v-col>
              </v-row>
            </div>
          </v-responsive>
        </v-container>
      </v-sheet>
      <v-overlay
      v-model="overlay"
      class="align-center justify-center"
      >
        <v-carousel
        hide-delimiters
        height="30%"
        >
          <v-carousel-item
          v-for="item in carousel_items"
          :key="item"
          >
            <v-container>
              <v-row>
                <v-col>
                  <v-responsive
                  width="100vw"
                  ></v-responsive>
                  <v-img
                  :src="server_static_url + item.path"
                  :lazy-src="`assets/images/mudori_lazy.png`"
                  class="mx-auto ml-auto"
                  min-width="30vw"
                  max-width="50vw"
                  min-height="30vh"
                  max-height="50vh"
                  aspect-ratio="1.5"
                  >
                  </v-img>
                  <v-spacer></v-spacer>
                  <div
                  class="black-hansans-font text-center pt-10"
                  ><span>{{ item.subtitle }}</span>
                  <v-spacer></v-spacer>
                  </div>
                  <div
                  class="text-center pt-10"
                  >
                  <v-btn
                  color="success"
                  @click="overlay = false"
                  >
                  <div class="jua-font">닫기</div></v-btn>
                  </div>
                </v-col>
              </v-row>
            </v-container>
          </v-carousel-item>
        </v-carousel>
      </v-overlay>
    </div>
</template>

<script>
  export default {
    data: () => ({
      person_items: ['유재석', '박명수', '정준하', '노홍철', '정형돈', '하하', '길', '전진', '황광희', '조세호', '양세형', '기타'],
      person_value: ['유재석', '박명수', '정준하', '노홍철', '정형돈', '하하', '길', '전진', '황광희', '조세호', '양세형', '기타'],
      emotion_checkbox: true,
      items: [],
      filtered_items: [],
      isLoading: false,
      isLoaded: false,
      overlay: false,
      carousel_items: [],
      url: '',
      server_url: 'http://18.181.188.181:3000/memes/',
      server_static_url: 'http://18.181.188.181:3000/images/',
      keyword: '',
    }),
    watch: {
      emotion_checkbox: 'filterItems',
      person_value: 'filterItems',
    },
    methods: {
      submit () {
        this.isLoading = true
        this.getList(this.keyword)
      },
      create_carousel_array (index) {
        if (index !== 0) {
          const begin = this.filtered_items.slice(index)
          const end = this.filtered_items.slice(0, index)
          this.carousel_items = begin.concat(end)
        } else {
          this.carousel_items = this.filtered_items
        }
      },
      open_overlay (index) {
        this.overlay = !this.overlay
        this.create_carousel_array(index)
      },
      async getList (keyword) {
        this.url = this.server_url + 'search?keyword=' + keyword
        const returnedList = await this.$api(this.url, 'get')
        const proxy = JSON.parse(JSON.stringify(returnedList))
        this.items = proxy.memes
        this.filtered_items = this.items
        this.filterItems()
        this.isLoading = false
        this.isLoaded = true
      },
      findCommonElements (arr1, arr2) {
        return arr1.some(item => arr2.includes(item))
      },
      filterItems () {
        if (this.emotion_checkbox) {
          this.filtered_items = this.items
            .filter(el => el.emotion_concord === this.emotion_checkbox)
        } else {
          this.filtered_items = this.items
        }
        this.filtered_items = this.filtered_items
          .filter(el => this.findCommonElements(el.personName, this.person_value))
      },
    },
  }
</script>

<style>
.black-hansans-font {
  font-size: xxx-large;
  color: white;
  font-family: 'BaeMinJUA', sans-serif !important;
}
.black-hansans-font span {
  background-color: black;
}
.v-application {
  font-family: 'BaeMinJUA', sans-serif !important;
}
.jua-font {
  font-family: 'BaeMinJUA', sans-serif !important;
}
.outlined {
  color: black;
}
@font-face {
  font-family:'BaeMinJUA';
  src: url('@/assets/fonts/BMJUA_ttf.ttf') format('truetype');
  font-weight: 100;
}
</style>
