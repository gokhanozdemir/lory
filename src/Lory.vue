<template>
  <div class="slider js_slider has_dots">
    <div class="frame js_frame">
      <ul class="slides js_slides">
        <slot></slot>
      </ul>
    </div>
    <slot name="actions"></slot>
    <ul ref="slider_dots" class="js_dots dots"></ul>
  </div>
</template>

<script>

// import { lory } from 'lory.js'
export default {
  props: {
    options: {
      type: Object,
      default: () => ({})
    }
  },

  data () {
    return {
      slider: null,
      dot_count: null,
      dot_container: null,
      dot_list_item: null
    }
  },

  mounted () {
    const lory = require('lory.js').lory // import lory in a ssr friendly way

    this.dot_count = this.$el.querySelectorAll('.js_slide').length
    this.dot_container = this.$refs['slider_dots']
    this.dot_list_item = document.createElement('li')
  
    this.slider = lory(this.$el, this.options)

    this.$el.addEventListener('before.lory.init', this.beforeSliderInit(this.slider, this.dot_container, this.dot_list_item));
    this.$el.addEventListener('after.lory.init', this.afterSliderInit(this.slider, this.dot_container));
    this.$el.addEventListener('after.lory.slide', this.afterLorySlide(this.slider, this.dot_container));
    this.$el.addEventListener('on.lory.resize', this.afterLoryResize);   
  },
  beforeDestroy () {
    this.$el.removeEventListener('before.lory.init', this.beforeSliderInit);
    this.$el.removeEventListener('after.lory.init', this.afterSliderInit);
    this.$el.removeEventListener('after.lory.slide', this.afterLorySlide);
    this.$el.removeEventListener('on.lory.resize', this.afterLoryResize);
    this.slider.destroy()
  },
  methods: {
    beforeSliderInit (slider, container, dot) {
        for (const i = 0; i < this.dot_count; i++) {
          const clone = dot.cloneNode();
          container.appendChild(clone);
        }
        container.childNodes[0].classList.add('active');
    },
    afterSliderInit (slider, container) {
      for (const i = 0; i < this.dot_count; i++) {
        container.childNodes[i].addEventListener('click', function(e) {
          slider.slideTo(Array.prototype.indexOf.call(container.childNodes, e.target));
        });
      }
    },
    afterLorySlide (slider, container) {
      return function (event) {
        for (const i = 0; i < 3; i++) {
          container.childNodes[i].classList.remove('active');
        }
        container.childNodes[event.detail.currentSlide - 1].classList.add('active');
      }
    },
    afterLoryResize (event) {
      let nodeLength = event.target.lastChild.childElementCount
      for (const i = 0; i < nodeLength; i++) {
        event.target.lastChild.childNodes[i].classList.remove('active');
      }
      event.target.lastChild.childNodes[0].classList.add('active');
    }
  }
}
</script>

<style lang="scss">
/**
 * (optional) define here the style definitions which should be applied on the slider container
 * e.g. width including further controls like arrows etc.
 */
.slider {
  .frame {
    /**
     * (optional) wrapper width, specifies width of the slider frame.
     */
    width: 100%;

    position: relative;
    font-size: 0;
    line-height: 0;
    overflow: hidden;
    white-space: nowrap;
  }

  .slides {
    width: 100%;
    display: inline-block;
  }

  li {
    position: relative;
    display: inline-block;

    /**
     * (optional) if the content inside the slide element has a defined size.
     */
    width: 100%;

    position: relative;
    display: inline-block;
    text-align: center;
    font-size: 15px;
    line-height: 30px;
  }

  .prev, .next {
    position: absolute;
    top: 50%;
    margin-top: -25px;
    display: block;
    cursor: pointer;
  }

  .next {
    right: 0;
  }

  .prev {
    left: 0;
  }

  .next svg, .prev svg {
    width: 25px;
  }

  &.js_rewind {
    .frame li {
      margin-right: 10px;
    }
  }

  // dots styling
  .dots {
    transform: translateY(-40px);
    text-align: center;
    position: absolute;
    width: 100%;
    z-index: 3;
  }

  .dots > li {
      background-color: gray;
      border-radius: 50%;
      cursor: pointer;
      display: inline-block;
      height: 12px;
      margin: 0 5px;
      width: 12px;
  }

  .dots > li.active {
    background-color: green;
  }
}
</style>
