/* 表情选择
 * @Author: MasonChen
 * @Date: 2018-09-27 13:50:11
 * @Last Modified by: MasonChen
 * @Last Modified time: 2018-11-20 15:12:15
 */
<template>
  <div class="face-container">
    <div class="scroll-wrapper">
      <div class="face-content"
        @touchmove.prevent
        ref="faceContent"
        :style="{ width: scrollWidth + 'px', marginLeft: -scrollPosition + 'px' }"
      >
        <div class="face-page"
          v-for="(page, index) in faces"
          :key="index">
          <a v-for="(item, j) in page"
            :key=j
            >
            <img @click="onFaceClick(`[${item.title}]`)" :src=item.src width="20" height="20">
          </a>
        </div>
      </div>
      <div class="page-dot">
        <div v-for="n in faces.length"
          @click="active = n - 1"
          :key=n class="dot-item"
          :class="n === (active + 1) ? 'active' : ''">
        </div>
      </div>
    </div>
    <div class="face-footer">
      <div class="footer-title">
        <a v-for="(faceList, index) in faceData"
          @click="onTypeClick(index)"
          :class="index === faceIndex ? 'active' : ''"
          :key=index
          >
          <img :src=faceList.faceList[0].src width="20" height="20">
        </a>
      </div>
    </div>
  </div>
</template>

<script>
import faceJSON from './face.json'
export default {
  data: () => ({
    active: 0,
    faceData: faceJSON,
    faceIndex: 0
  }),
  computed: {
    faces: function () {
      const { faceData, faceIndex } = this
      const list = faceData[faceIndex].faceList
      let n = 0
      let listGrouped = []
      while (n * 21 < list.length) {
        listGrouped.push(list.slice(n * 21, (n + 1) * 21))
        n += 1
      }
      return listGrouped
    },
    scrollWidth: function () {
      return this.faces.length * document.body.scrollWidth
    },
    scrollPosition: function () {
      return this.active * document.body.scrollWidth
    }
  },
  mounted () {
    const el = this.$refs.faceContent
    let x1 = 0
    let x2 = 0
    el.ontouchstart = (evt) => {
      x1 = evt.targetTouches[0].pageX
    }
    el.ontouchend = (evt) => {
      x2 = evt.changedTouches[0].pageX
      if (x2 - x1 > 50) {
        this.active !== 0 && this.active--
      } else if (x2 - x1 < -50) {
        this.active !== this.faces.length - 1 && this.active++
      }
    }
  },
  methods: {
    onTypeClick (index) {
      this.faceIndex = index
      this.active = 0
    },
    onFaceClick (face) {
      this.$emit('onFaceChoose', face)
    }
  }
}
</script>

<style lang="scss" scoped>
.face-container {
  width: 100%;
  background: #F6F6F6;
  display: flex;
  flex-direction: column;
  .scroll-wrapper {
    width: 100%;
    height: 157px;
    border-top: solid 1px #DDDDDD;
    position: relative;
    .face-content {
      height: 100%;
      display: flex;
      transition: margin-left .5s;
      .face-page {
        width: 100%;
        height: 140px;
        display: flex;
        flex-wrap: wrap;
        align-content: flex-start;
        a {
          width: 14.28%;
          height: 33.33%;
          display: flex;
          justify-content: center;
          align-items: center;
        }
      }
    }
    .page-dot {
      position: absolute;
      bottom: 10px;
      width: 100%;
      display: flex;
      justify-content: center;
      .dot-item {
        width: 6px;
        height: 6px;
        border-radius: 50%;
        background: #CCCCCC;
        margin: 0 3px;
      }
      .dot-item.active {
        background: #FF8140;
      }
    }
  }
  .face-footer {
    position: relative;
    width: 100%;
    height: 36px;
    overflow-x: auto;
    .footer-title {
      width: 280px;
      height: 100%;
      overflow-x: auto;
      display: flex;
      flex-wrap: wrap;
      align-items: center;
      a {
        height: 100%;
        width: 56px;
        display: flex;
        align-items: center;
        justify-content: center;
      }
      a.active {
        background: #E9E9E9;
      }
    }
  }
}
</style>
<style lang="scss">
.face-content .v-tabs__wrapper {
  display: none;
}
</style>
