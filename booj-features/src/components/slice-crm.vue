<template>
  <div class="sliceCRM">
    <div class="container slide-wrapper">
      <div class="top-text">
        <div class="title">{{title}}</div>
        <div class="description">{{description}}</div>
      </div>
      
      <div class="galleryWrap">
        <carousel-3d ref="carousel" :perspective="carouOpts.perspective" :space="carouOpts.space" :display="carouOpts.display" :controls-visible="carouOpts.controlsVisible" :width="carouOpts.width" :height="carouOpts.height" @before-slide-change="onBeforeSlideChange" @after-slide-change="onAfterSlideChange">
          <slide v-for="(slide, index) in gallerySlides" v-bind:key="index" :index="index" >
            <prismic-image v-if="slide.graphic" :field="slide.graphic"/>
          </slide>
        </carousel-3d>
        <div class="galleryBg"></div>
        <div class="galleryNextBtn" v-on:click="galNextSlide"><img src="@/assets/cta-arrow.svg">{{galleryNextTitle}}</div>
        <div class="galleryPrevBtn" v-on:click="galPrevSlide">{{galleryPrevTitle}}<img src="@/assets/cta-arrow.svg"></div>
        <div class="galleryNextBtnMobile" v-on:click="galNextSlide">
          <img src="@/assets/cta-arrow.svg">
        </div>  
        <div class="galleryPrevBtnMobile" v-on:click="galPrevSlide">
          <img src="@/assets/cta-arrow.svg">
        </div>  
      </div>
      <div id="slideInfo">
        <div class="slideInfoWrapper slideInfoPre" v-bind:class="[displayPreSlide ? 'active' : '']">
          <div class="h1 mobile-only">{{slideInfo.slide_namePre}}</div>
          <hr>
          <div class="bbTitle">{{slideInfo.titlePre}}</div>
          <div class="bbDescritpion">
            <prismic-rich-text class="pre-tooltip-text" v-if="slideInfo.descriptionPre" :field="slideInfo.descriptionPre" />
            <div class="tourButton" v-if="slideInfo.tourTextPre && slideInfo.tourLinkPre"><a :href="slideInfo.tourLinkPre.url">{{slideInfo.tourTextPre}}</a></div>
          </div>
        </div>
        <div class="slideInfoWrapper slideInfoPost" v-bind:class="[displayPreSlide ? '' : 'active']">
          <div class="h1 mobile-only">{{slideInfo.slide_namePost}}</div>
          <hr>
          <div class="bbTitle">{{slideInfo.titlePost}}</div>
          <div class="bbDescritpion">
            <prismic-rich-text class="pre-tooltip-text" v-if="slideInfo.descriptionPost" :field="slideInfo.descriptionPost" />
            <div class="tourButton" v-if="slideInfo.tourTextPost && slideInfo.tourLinkPost"><a :href="slideInfo.tourLinkPost.url">{{slideInfo.tourTextPost}}</a></div>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import { Carousel3d, Slide } from 'vue-carousel-3d';

export default {
  props: ['title', 'description', 'gallerySlides'],
  data () {
    return {
      galleryNextTitle: "Next",
      galleryPrevTitle: "Prev",
      displayPreSlide: true,
      slideInfo: {
        titlePre: null,
        descriptionPre: null,
      },
      carouOpts: {
        space: 350,
        display: 3,
        perspective: 0,
        width: 360,
        height: 220,
        controlsVisible: false
      }
    }
  },
  methods: {
    getCommonSlideIndex () {
      let currSlideIndex = this.$refs.carousel.currentIndex
      let prevSlideIndex = (currSlideIndex === 0) ? this.gallerySlides.length - 1 : currSlideIndex - 1
      let nextSlideIndex = (currSlideIndex === this.gallerySlides.length - 1) ? 0 : currSlideIndex + 1

      return {curr: currSlideIndex, prev: prevSlideIndex, next: nextSlideIndex}
    },
    galNextSlide () {
      this.$refs.carousel.goNext()
    },
    galPrevSlide () {
      this.$refs.carousel.goPrev()
    },
    setBaseInfoWindow () {
      let csi = this.getCommonSlideIndex()
      let currSlideIndex = csi.curr

      this.slideInfo.titlePre = this.gallerySlides[currSlideIndex].blue_box_title
      this.slideInfo.descriptionPre = this.gallerySlides[currSlideIndex].blue_box_description
      this.slideInfo.tourTextPre = this.gallerySlides[currSlideIndex].blue_box_button_text
      this.slideInfo.tourLinkPre = this.gallerySlides[currSlideIndex].blue_box_button_link
      this.slideInfo.slide_namePre = this.gallerySlides[currSlideIndex].slide_name
      this.slideInfo.tourPagePre = this.gallerySlides[currSlideIndex].tour_page
    },
    onBeforeSlideChange () {
      let csi = this.getCommonSlideIndex()
      let currSlideIndex = csi.curr

      this.displayPreSlide = !this.displayPreSlide
      if(this.displayPreSlide){
        this.slideInfo.titlePre = this.gallerySlides[currSlideIndex].blue_box_title
        this.slideInfo.descriptionPre = this.gallerySlides[currSlideIndex].blue_box_description
        this.slideInfo.tourTextPre = this.gallerySlides[currSlideIndex].blue_box_button_text
        this.slideInfo.tourLinkPre = this.gallerySlides[currSlideIndex].blue_box_button_link
        this.slideInfo.slide_namePre = this.gallerySlides[currSlideIndex].slide_name
        this.slideInfo.tourPagePre = this.gallerySlides[currSlideIndex].tour_page
      }else{
        this.slideInfo.titlePost = this.gallerySlides[currSlideIndex].blue_box_title
        this.slideInfo.descriptionPost = this.gallerySlides[currSlideIndex].blue_box_description
        this.slideInfo.tourTextPost = this.gallerySlides[currSlideIndex].blue_box_button_text
        this.slideInfo.tourLinkPost = this.gallerySlides[currSlideIndex].blue_box_button_link
        this.slideInfo.slide_namePost = this.gallerySlides[currSlideIndex].slide_name
        this.slideInfo.tourPagePost = this.gallerySlides[currSlideIndex].tour_page
      }
    },
    onAfterSlideChange () {
      //window.console.log('sooooo', this.$refs.carousel, this.gallerySlides[index])
      this.setCurrSlideOpts()
    },
    setCurrSlideOpts () {
      let csi = this.getCommonSlideIndex()
      //let currSlideIndex = csi.curr
      let prevSlideIndex = csi.prev
      let nextSlideIndex = csi.next

      this.galleryNextTitle = this.gallerySlides[nextSlideIndex].slide_name
      this.galleryPrevTitle = this.gallerySlides[prevSlideIndex].slide_name
    },
    responsiveOpts () {
      let windowWidth = window.outerWidth
      if(windowWidth > 1024){
        this.carouOpts.space = 500
        this.carouOpts.width = 500
        this.carouOpts.height = 300
      }else if(windowWidth > 768){
        this.carouOpts.space = 450
        this.carouOpts.width = 405
        this.carouOpts.height = 250
      }else{
        this.carouOpts.space = 350
        this.carouOpts.width = 360
        this.carouOpts.height = 220
      }
    }
  },
  mounted () {
    this.setBaseInfoWindow()
    this.setCurrSlideOpts()
    this.responsiveOpts()
    window.addEventListener('resize', this.responsiveOpts)
  },
  components: {
    Carousel3d,
    Slide
  }
}
</script>