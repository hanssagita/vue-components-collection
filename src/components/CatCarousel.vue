<template>
  <div>
    <div class="cat-carousel-container">
      <div
        :class="['cat-carousel__navigation', {'cat-carousel__navigation--end': onFirstPage}]"
      >
        <template>
          <slot name="prev-navigation" :prev="prev">
            <div class="cat-carousel__default-nav cat-carousel__default-nav--left" @click="prev">
              <img src="https://i.imgur.com/PpHTPrc.png">
            </div>
          </slot>
        </template>
      </div>
      <div
        ref="carouselContent"
        class="cat-carousel__content"
      >
        <div
          ref="carouselWrapper"
          :style="wrapperStyles"
          class="cat-carousel__content__wrapper"
          @touchstart="touchStart"
          @touchmove="touchMove"
        >
          <div
            v-for="(item, index) in items"
            ref="carouselItem"
            :key="index"
            class="cat-carousel__content__wrapper__item"
            :style="carouselItemStyles"
          >
            <template>
              <slot
                name="item"
                :data="item"
                :index="index"
              />
            </template>
          </div>
        </div>
      </div>
      <div
        :class="['cat-carousel__navigation cat-carousel__navigation__next',
                 {'cat-carousel__navigation--end': onLastPage}]"
      >
        <template>
          <slot name="next-navigation" :next="next">
            <div class="cat-carousel__default-nav" @click="next">
              <img src="https://i.imgur.com/PpHTPrc.png">
            </div>
          </slot>
        </template>
      </div>
    </div>
    <div
      :class="{'hide': hideIndicators}"
      class="cat-carousel__indicators"
    >
      <div
        v-for="index in maxSlide"
        :key="index"
        :class="['cat-carousel__indicators__item']"
        :style="[indicatorsItemSizeStyle, selectedIndicator(index) && activeIndicatorStyle]"
      />
    </div>
  </div>
</template>

<script>
  const WIDTH_PAGE = 100
  const SWIPE_THRESHOLD = 80
  const CENTER_MODE_DEFAULT_CONFIG = {
    enabled: false,
    paddingCenter: 10
  }
  const INDICATORS_DEFAULT_CONFIG = {
    size: 16,
    color: '#d6d6d6',
    activeColor: '#0095da',
    hideIndicators: false
  }

  export default {
    name: 'CatCarousel',
    props: {
      items: {
        type: Array,
        default: () => [],
        required: true
      },
      itemPerPage: {
        type: Number,
        default: 5
      },
      indicatorsConfig: {
        type: Object,
        default: () => {
          return INDICATORS_DEFAULT_CONFIG
        }
      },
      centerMode: {
        type: Object,
        default: () => {
          return CENTER_MODE_DEFAULT_CONFIG
        }
      }
    },
    data () {
      return {
        itemWidth: 0,
        wrapper: {
          translateX: 0
        },
        maxSlide: 0,
        track: 0,
        slides: [],
        normalSlideWindow: [],
        reversedSlideWindow: [],
        touchX: null
      }
    },
    mounted () {
      this.maxSlide = Math.ceil(this.items.length / this.itemPerPage)
      this.itemWidth = this.carouselItem.length > 0 && this.carouselItem[0].clientWidth
      this.initSlides()
    },
    watch: {
      items () {
        this.wrapper = {
          translateX: 0
        }
        this.track = 0
        this.maxSlide = Math.ceil(this.items.length / this.itemPerPage)
        this.itemWidth = this.carouselItem.length > 0 && this.carouselItem[0].clientWidth
        this.initSlides()
      }
    },
    computed: {
      carouselContent () {
        return this.$refs.carouselContent
      },
      carouselItem () {
        return this.$refs.carouselItem
      },
      carouselWrapper () {
        return this.$refs.carouselWrapper
      },
      wrapperStyles () {
        if (this.centerMode.enabled) {
          return {transform: `translateX(calc(${this.wrapper.translateX}px + ${this.centerMode.paddingCenter}%)`}
        }

        return {transform: `translateX(${this.wrapper.translateX}px)`}
      },
      onFirstPage () {
        return this.track === 0
      },
      onLastPage () {
        return this.track === this.maxSlide - 1
      },
      carouselItemStyles () {
        let width = WIDTH_PAGE / this.itemPerPage

        if (this.centerMode.enabled) {
          width = width * (1 - this.centerMode.paddingCenter / 100 * 2)
        }

        return {
          flex: `0 0 ${width}%`,
          width: `${width}%`
        }
      },
      indicatorsItemSizeStyle () {
        const size = this.indicatorsConfig.size || INDICATORS_DEFAULT_CONFIG.size
        return {
          width: `${size}px`,
          height: `${size}px`,
          backgroundColor: this.indicatorsConfig.color || INDICATORS_DEFAULT_CONFIG.color
        }
      },
      activeIndicatorStyle () {
        return {
          backgroundColor: this.indicatorsConfig.activeColor || INDICATORS_DEFAULT_CONFIG.activeColor
        }
      },
      hideIndicators () {
        return this.indicatorsConfig.hideIndicators || INDICATORS_DEFAULT_CONFIG.hideIndicators
      }
    },
    methods: {
      initSlides () {
        this.slides = this.addSlides(this.items.length)
        this.initialSlides = this.slides
        this.reversedSlides = this.slides.slice().reverse()
      },
      addSlides (itemsLength) {
        if (itemsLength <= 0) return []
        const count = Math.min(itemsLength, this.itemPerPage)
        let slides = []
        slides = slides.concat([count], this.addSlides(itemsLength-count))
        return slides
      },
      prev () {
        if (this.onFirstPage) return
        this.track--
        this.wrapper = Object.assign({}, this.wrapper,{
          translateX: this.wrapper.translateX + this.slides[this.track] * this.itemWidth
        })
        if (this.onFirstPage) this.slides = this.initialSlides
      },
      next () {
        if (this.onLastPage) return
        this.track++
        this.wrapper = Object.assign({}, this.wrapper, {
          translateX: this.wrapper.translateX - this.slides[this.track] * this.itemWidth
        })
        if (this.onLastPage) this.slides = this.reversedSlides
      },
      selectedIndicator (index) {
        return index === this.track + 1
      },
      touchStart (event) {
        this.touchX = event.touches[0].clientX
      },
      touchMove (event) {
        if (!this.touchX) return
        let currentX = event.touches[0].clientX
        let diffX = currentX - this.touchX
        if (diffX > SWIPE_THRESHOLD) {
          this.prev()
          this.touchX = null
        }
        if (diffX < -SWIPE_THRESHOLD) {
          this.next()
          this.touchX = null
        }
      }
    }
  }

</script>

<style lang="scss" scoped>
  .cat-carousel-container {
    display: flex;
    position: relative;
    width: 100%;
    height: 100%;
    margin: 0 auto;
  }
  .cat-carousel {
    &__content {
      width: 100%;
      overflow: hidden;
      display: inline-block;
      &__wrapper {
        transition: transform 0.5s ease-out;
        display: flex;
        align-items: flex-start;
        height: 100%;
        &__item {
          box-sizing: border-box;
          padding: 0 1%;
          display: flex;
          height: 100%;
        }
      }
    }
    &__navigation {
      display: flex;
      align-self: center;
      position: absolute;
      z-index: 2;
      &__next {
        right: 0;
      }
      &--end {
        display: none;
      }
    }
    &__default-nav {
      width: 48px;
      height: 48px;
      opacity: 0.88;
      box-shadow: 0 0 24px -4px rgba(0, 0, 0, 0.24);
      background-color: #ffffff;
      display: flex;
      justify-content: center;
      align-content: center;
      align-items: center;
      border-radius: 50%;
      border-right: 1px solid #eee;
      cursor: pointer;
      z-index: 2;
      @media only screen and (max-width: 768px) {
        display: none;
      }
      img {
        width: 34px;
        height: 34px;
      }
      &--left {
        img {
          transform: rotate(180deg);
        }
      }
    }
    &__indicators {
      display: flex;
      justify-content: center;
      margin: 8px;
      &__item {
        border-radius: 50%;
        margin: 0 4px;
      }
    }
  }
  .hide {
    display: none;
  }
</style>
