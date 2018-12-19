<template>
  <div id="app">
    <p style="padding-left: 20px;">请在手机模拟器模式下预览 ：）</p>
    <div class="detail-footer">
      <textarea placeholder="评论" v-model="shareText" ref="shareTextArea" @focus="showFacePanel = false"></textarea>
      <div class="footer-hint">10-140字</div>
      <div class="footer-submit">
        <div>
          <img :src="IconSmile" v-if="!showFacePanel" @click="showFacePanel = true"/>
          <img :src="IconBoard" v-if="showFacePanel" @click="handleKeyboardClick"/>
        </div>
        <button>
          提交
        </button>
      </div>
      <emoji-input-method @onFaceChoose="handleFaceChoose" v-if="showFacePanel"></emoji-input-method>
    </div>
  </div>
</template>

<script>
import IconSmile from './assets/images/icon_smile.png'
import IconBoard from './assets/images/icon_keyboard.png'
import EmojiInputMethod from './components/EmojiInputMethod'
export default {
  name: 'app',
  components: {
    EmojiInputMethod
  },
  data () {
    return {
      IconSmile,
      IconBoard,
      shareText: '',
      showFacePanel: true
    }
  },
  methods: {
    handleFaceChoose (face) {
      const value = this.shareText
      const el = this.$refs.shareTextArea
      const startPos = el.selectionStart
      const endPos = el.selectionEnd
      const newValue =
        value.substring(0, startPos) +
        face +
        value.substring(endPos, value.length)
      this.shareText = newValue
      if (el.setSelectionRange) {
        setTimeout(() => {
          const index = startPos + face.length
          el.setSelectionRange(index, index)
        }, 0)
      }
    },
    handleKeyboardClick () {
      this.showFacePanel = false
      this.$refs.shareTextArea.focus()
    }
  }
}
</script>

<style lang="scss" scoped>
.detail-footer {
  background: #F9F9F9;
  position: fixed;
  bottom: 0;
  width: 100%;
  textarea {
    box-sizing: border-box;
    height: 92px;
    width: calc(100% - 16px);
    margin: 8px;
    resize: none;
    font-size: 14px;
    background: #FFFFFF;
    border: 1px solid #D9D9D9;
    border-radius: 4px;
  }
  .footer-hint {
    position: absolute;
    right: 15px;
    top: 78px;
    font-size: 12px;
    color: #BBBBBB;
    letter-spacing: 0;
  }
  img {
    width: 24px;
    height: 24px;
    margin-left: 14px;
  }
  .footer-submit {
    display: flex;
    align-items: center;
    justify-content: space-between;
    color: #BBBBBB;
    padding-bottom: 8px;
    :nth-child(1) {
      display: flex;
      align-items: center;
    }
    span {
      margin-left: 16px;
      font-size: 12px;
    }
    button {
      color: #fff;
      background-color: #108ee9;
      border-color: #108ee9;
      border: 1px solid transparent;
      width: 70px;
      height: 30px;
      margin-right: 8px;
      font-size: 12px;
    }
  }
}
</style>
<style>
body {
  margin: 0;
}
</style>
