<template lang='pug'>
  transition(name='modal')
    .modal-slideshow(@click='close', v-if='imgIndex !== null')
      .modal-slideshow__close(@click="close") x
      button.modal-slideshow__prev(@click.stop="onPrev")
      .modal-slideshow__container(@click.stop="onNext", v-if="images")
        .modal-slideshow__container__image
          img.modal-slideshow__container__image__img(@click.stop="onNext", :src="imageUrl")
      button.modal-slideshow__next(@click.stop="onNext")
      .modal-slideshow__gallery(ref="gallery")
        .modal-slideshow__gallery__title(v-if="images") {{ imgIndex + 1 }} / {{ images.length }}
        .modal-slideshow__gallery__container(
        v-if="images",
        :style="{ transform: 'translate(' + galleryXPos + 'px, 0)' }"
        )
          img.modal-slideshow__gallery__container__img(
          v-for="(image, i) in images", :src="image",
          @click.stop="onClickThumb(image, i)",
          :class="{ 'modal-slideshow__gallery__container__img--active': i === imgIndex}"
          )
</template>

<script>
  export default {
    props: ['images', 'index'],
    mounted() {
      window.addEventListener('keydown', e => {
        if (e.keyCode === 37) {
          this.onPrev()
        }

        if (e.keyCode === 39) {
          this.onNext()
        }
      })
    },
    watch: {
      index(val) {
        this.imgIndex = val
      }
    },
    methods: {
      close() {
        this.imgIndex = null
      },
      onPrev() {
        if (this.imgIndex > 0) {
          this.imgIndex--
        } else {
          this.imgIndex = this.images.length - 1
        }
        this.updateThumbails()
      },
      onNext() {
        if (this.imgIndex < this.images.length - 1) {
          this.imgIndex++
        } else {
          this.imgIndex = 0
        }
        this.updateThumbails()
      },
      onClickThumb(image, index) {
        this.imgIndex = index
        this.updateThumbails()
      },
      updateThumbails() {
        if (!this.$refs.gallery) {
          return
        }

        const galleryWidth = this.$refs.gallery.clientWidth
        const currThumbsWidth = this.imgIndex * this.thumbnailWidth
        const centerPos = Math.floor(galleryWidth / (this.thumbnailWidth * 2)) * this.thumbnailWidth

        if (currThumbsWidth < centerPos || galleryWidth >= currThumbsWidth) {
          this.galleryXPos = 0
        } else if (currThumbsWidth > (this.images.length * this.thumbnailWidth) - galleryWidth + centerPos) {
          this.galleryXPos = -(this.images.length * this.thumbnailWidth - galleryWidth - 20)
        } else {
          this.galleryXPos = -(this.imgIndex * this.thumbnailWidth) + centerPos
        }
      }
    },
    computed: {
      imageUrl() {
        return this.images[this.imgIndex]
      },
    },
    data() {
      return {
        imgIndex: this.index,
        image: null,
        galleryXPos: 0,
        thumbnailWidth: 120
      }
    }
  }
</script>

<style lang="scss">
  $black-alpha-80: rgba(0, 0, 0, 0.8);
  $black: #000;
  $white: #fff;
  $radius-medium: 8px;
  $radius-large: 12px;

  @mixin respond-to($breakpoint) {
    @if $breakpoint == "small" {
      @media (min-width: 767px) {
        @content;
      }
    } @else if $breakpoint == "medium" {
      @media (min-width: 992px) {
        @content;
      }
    } @else if $breakpoint == "large" {
      @media (min-width: 1200px) {
        @content;
      }
    }
  }

  @mixin modal-base() {
    transition: opacity 0.2s ease;
    position: fixed;
    z-index: 9998;
  }

  @mixin modal-mask() {
    @include modal-base();

    top: 0;
    left: 0;
    width: 100%;
    min-height: 100%;
    height: 100vh;
    background-color: $black-alpha-80;
    display: table;
  }

  .modal-slideshow {
    @include modal-mask();
    background-color: $black-alpha-80;

    &__close {
      color: #fff;
      position: absolute;
      top: 10px;
      right: 10px;
      pointer: cursor;
    }

    &__prev,
    &__next {
      position: absolute;
      top: 50%;
      margin-top: -25px;
      width: 50px;
      height: 50px;
      background-repeat: no-repeat;
      background-position: center;
      z-index: 999;
      cursor: pointer;
    }

    &__prev {
      left: 0;
      margin-top: -25px;
      //background-image: url('/images/arrow_white_left.svg');
    }

    &__next {
      right: 0;
      //background-image: url('/images/arrow_white_right.svg');
    }

    &__container {
      position: absolute;
      cursor: pointer;
      margin: 0 auto;
      overflow: hidden;

      /*
      @include respond-to(small) {
        width: 100%;
        top: 50%;
        margin-top: -150px;
      }

      @include respond-to(medium) {
        max-width: 100vh;
        margin: auto;
        top: 2rem;
        left: 0.5rem;
        right: 0.5rem;
      }
      */

      max-width: 100vh;
      margin: auto;
      top: 2rem;
      left: 0.5rem;
      right: 0.5rem;

      &__image {
        background-color: $black;

        /*
        @include respond-to(small) {
          height: 274px;
        }

        @include respond-to(medium) {
          height: 60vh;
          border-radius: $radius-large;
          overflow: hidden;
        }
        */

        height: 60vh;
        border-radius: $radius-large;
        overflow: hidden;

        &__img {
          display: block;
          margin: 0 auto;
          height: 100%;
        }
      }
    }
  }

  .modal-slideshow__gallery {
    /*
    @include respond-to(small) {
      display: none;
    }

    @include respond-to(medium) {
      overflow-x: hidden;
      overflow-y: hidden;
      position: absolute;
      bottom: 10px;
      margin: auto;
      max-width: 100vh;
      white-space: nowrap;
      left: 0.5rem;
      right: 0.5rem;
    }
    */

    overflow-x: hidden;
    overflow-y: hidden;
    position: absolute;
    bottom: 10px;
    margin: auto;
    max-width: 100vh;
    white-space: nowrap;
    left: 0.5rem;
    right: 0.5rem;

    &__title {
      color: $white;
      margin-bottom: 0.5rem;
    }

    &__container {
      overflow: visible;
      display: block;
      height: 100px;
      white-space: nowrap;
      transition: all 200ms ease-in-out;
      width: 100%;

      &__img {
        width: 100px;
        height: 100px;
        object-fit: cover;
        display: inline-block;
        float: none;
        margin-right: 20px;
        cursor: pointer;
        opacity: 0.6;
        border-radius: $radius-medium;
      }

      &__img--active {
        width: 100px;
        display: inline-block;
        float: none;
        opacity: 1;
      }
    }
  }

</style>