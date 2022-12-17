<template>
  <div ref="container" id="container" contenteditable="true">{{ text }}</div>
</template>

<script>
import { ref } from 'vue'
export default {
  name: 'EditorText',
  setup() {
    const count = ref(0)
    const selectData = ref({
      anchorOffset: 0,
      focusOffset: 0
    })
    const text = ref('')

    // 返回值会暴露给模板和其他的选项式 API 钩子
    return {
      count,
      selectData,
      text
    }
  },

  mounted() {
    console.log('this.$refs.container', this.$refs, this.$refs.container)
    this.$refs.container.addEventListener('beforeinput', async (e) => {
      // console.log(e)
      const inputType = e.inputType
      const container = this.$refs.container
      
      e.preventDefault()
      if (inputType === 'insertText') {
        let data = e.data
        if (data === '9') {
          data = '九'
        }
        
        this.text = this.text.slice(0, this.selectData.anchorOffset) + data + this.text.slice(this.selectData.anchorOffset)

        this.selectData.anchorOffset += data.length
        
        await this.$nextTick()
        const range = document.createRange()
        range.setStart(container.childNodes[0], this.selectData.anchorOffset)
        range.setEnd(container.childNodes[0], this.selectData.anchorOffset)
        window.getSelection().removeAllRanges()
        window.getSelection().addRange(range)
      }
      if (inputType === 'deleteContentBackward') {
        if (this.selectData.anchorOffset === this.selectData.focusOffset && this.selectData.focusOffset) {
          this.text = this.text.slice(0, this.selectData.anchorOffset - 1) + this.text.slice(this.selectData.anchorOffset)
        } else {
          this.text = this.text.slice(0, this.selectData.anchorOffset) + this.text.slice(this.selectData.focusOffset)
        }
      }
    })

    document.addEventListener('selectionchange', () => {
      const selection = window.getSelection()
      console.log('selection', selection)
      this.selectData.anchorOffset = selection.anchorOffset
      this.selectData.focusOffset = selection.focusOffset
    })
  }
}
</script>

<style scoped>
#container {
  background-color: #c1c1c1;
}
</style>
