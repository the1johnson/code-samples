<template>
  <div id="fullpagewrapper" :class="[ introClosed ? 'introclosed' : ' ' ]">
    <full-page v-if="this.fields.homeSlice" ref="fullpage" v-bind:options="this.fields.fpopts">
      <div class="section fp-auto-height height-none"></div>
      <div v-for="(slice, index) in this.fields.homeSlice" v-bind:key="index" v-bind:class="slice.primary.background_image" class="section fp-auto-height-responsive">
        <SliceSimple v-if="slice.slice_type === 'simple_slide'" id="Home" v-bind:graphic="slice.primary.graphic" v-bind:title="slice.primary.title" v-bind:description="slice.primary.description" />

        <SliceApp v-else-if="slice.slice_type === 'app_slide'" v-bind:graphic="slice.primary.graphic" v-bind:title="slice.primary.title" v-bind:description="slice.primary.description" v-bind:contentRightTitle="slice.primary.content_right_title" v-bind:contentRightImage="slice.primary.content_right_image" v-bind:contentRightDescription="slice.primary.content_right_description" v-bind:appleStoreLink="slice.primary.apple_store_link" v-bind:googleStoreLink="slice.primary.google_store_link" v-bind:buttonTxt="slice.primary.button_text" v-bind:buttonLink="slice.primary.button_link" />

        <SliceList v-else-if="slice.slice_type === 'list_slide'" v-bind:graphic="slice.primary.graphic" v-bind:title="slice.primary.title" v-bind:description="slice.primary.description" v-bind:preListDescription="slice.primary.pre_list_description" v-bind:buttonTxt="slice.primary.button_text" v-bind:buttonLink="slice.primary.button_link" v-bind:checkList="slice.items" />

        <SliceCRM v-else-if="slice.slice_type === 'crm_slide'" v-bind:gallerySlides="slice.items" v-bind:class="slice.primary.background_image" v-bind:title="slice.primary.top_title" v-bind:description="slice.primary.top_description" />

        <SliceSupport v-else-if="slice.slice_type === 'support_slide'" v-bind:title="slice.primary.title" v-bind:description="slice.primary.description" v-bind:whiteBoxTitleTop="slice.primary.white_box_title_top" v-bind:whiteBoxDescriptionTop="slice.primary.white_box_description_top" v-bind:whiteBoxTitleBottom="slice.primary.white_box_title_bottom" v-bind:whiteBoxDescriptionBottom="slice.primary.white_box_description_bottom" v-bind:buttonLink="slice.primary.button_link" v-bind:buttonTxt="slice.primary.button_text" v-bind:checkList="slice.items" />
      </div>
      <div class="section fp-auto-height last-section">
        <FooterComponent id="footer" />
      </div>
    </full-page>
    <div class="side-logo">
      <img src="@/assets/remax_logo_line.svg">
    </div>
  </div>
</template>

<script>
import SliceSimple from './slice-simple.vue'
import SliceApp from './slice-app.vue'
import SliceList from './slice-list.vue'
import SliceCRM from './slice-crm.vue'
import SliceSupport from './slice-support.vue'
import FooterComponent from './footer.vue'
import { eBus } from '../main'

export default {
  data () {
    return {
      introClosed: false,
      fields: {
        homeSlice: null,
        fpopts: {
          licenseKey: 'DBE56275-4E7F4563-B1EC41D0-77DDCB26',
          navigation: true,
          navigationPosition: 'right',
          // navigationTooltips: ['First page', 'Second page', 'Third and last page'],
          responsiveWidth: 1200,
          scrollingSpeed: 2000,
          autoScrolling: true,
          afterLoad: this.afterLoad,
        }
      }
    }
  },
  methods: {
    getContent () {
      this.$prismic.client.getSingle('homepage')
        .then((document) => {
          this.fields.homeSlice = document.data.body
        })
    },
    afterLoad() {   
      // if (element.classList.contains('active')) {
      //   alert('hi')
      // }
      var element = document.querySelectorAll('.section').length
      var bodyclass = document.querySelectorAll('.fp-viewing-'+(element-1))
      if (bodyclass.length == 1) {
          document.body.classList.add("last");
      } else {
          document.body.classList.remove("last");
      }

     // if(index == el.length){
     //  window.console.log('last')
     // }
    },
  },
  created () {
    this.getContent();
    eBus.$on('introHideUpdated', (data) => {
      if(data){
        this.$refs.fullpage.api.silentMoveTo(2)
        this.$refs.fullpage.api.setScrollingSpeed(1000)
        this.introClosed = true
      }
    })
  },
  components: {
    SliceSimple,
    SliceApp,
    SliceList,
    SliceCRM,
    SliceSupport,
    FooterComponent
  }
}
</script>