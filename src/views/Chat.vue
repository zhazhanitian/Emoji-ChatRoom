<template>
  <div id="chat" class="chat_page_con">
    
    <!-- 输入框 -->
    <div>Ctrl + Enter 发送消息：</div>
    <div 
      id="charInput" 
      @click="saveRangeLocal"
      @focus="saveRangeLocal" 
      @keyup="inputSend"
      @input="saveRangeLocal" 
      class="chatframe_input_con scrollbar" 
      contenteditable="true">
    </div>

    <!-- 表情选择器 -->
    <div class="chatframe-icon">
      <el-popover
        placement="bottom-start"
        width="400"
        trigger="click">
        <el-tabs tab-position="bottom" value="Emotions" class="emoji_tabs_box">
          <el-tab-pane 
            v-for="(emojiCon, emojiKey, eInd) in emojiIcon" 
            :key="emojiKey"
            :label="emojiKey" 
            :name="emojiKey">

            <!-- 这里的label icon不能放到json配置文件中，因为icon放到配置文件中后无法渲染出来 -->
            <!-- 如果想优化这里，可以根据自己的需求，试用图片来替换，使用同样的图片加载方法保存在json中 -->
            <span v-if="eInd == 0"  slot="label" class="iconfont emoji_pane_tab">&#xe612;</span>
            <span v-if="eInd == 1"  slot="label" class="iconfont emoji_pane_tab">&#xe609;</span>
            <span v-if="eInd == 2"  slot="label" class="iconfont emoji_pane_tab">&#xe62d;</span>
            <span v-if="eInd == 3"  slot="label" class="iconfont emoji_pane_tab">&#xe63c;</span>
            <img 
              v-for="(emoItem, emoInd) in emojiCon" 
              :key="emoInd" 
              :src="getIconPic(emoItem.unicode)" 
              alt="X"
              @click="sendEmojiIcon(emoItem.unicode)"
              class="chat_emoji_item">
          </el-tab-pane>
        </el-tabs>
        <el-button class="iconfont open_emoji_icon" type="text" slot="reference">&#xe612;</el-button>
      </el-popover>
    </div>

    <!-- 显示内容区 -->
    <div>发送的消息：</div>
    <div class="chatframe-text text_emoji" v-html="changeEmojiCon(sendValue)"></div>
  </div>
</template>


<script>
let emojiData = require('./../assets/emoji/emoji.json')

export default {
  name: 'chat',

  data () {
    return {
      emojiIcon: emojiData.icon,    // 导入的emoji表情配置文件内容
      emojiPath: new Map(),         // emoji表情地址map对象,
      inputRange: '',               // 光标
      sendValue: '',                // 发出的内容
    }
  },


  methods: {

    // 发送消息
    inputSend (e) {
      if ((e.ctrlKey && e.keyCode == 13) || (e.ctrlKey && e.keyCode == 108)) {
        // 提交的时候使用正则替换，将br换位换行符，不能使用innertext转，有兼容性问题
        this.sendValue = this.formatInputCon().replace(/<br>/g, '\r\n')
      } 
    },


    // 编译需要提交的内容
    HTMLDecode (text) { 
      let temp = document.createElement("div"); 
      temp.innerHTML = text
      var output = temp.innerText
      temp = null
      return output
    },


    // 延时记录光标到位置
    chartFramRange (type) { 
      try {
        if (type === 'click') {
          this.inputRange = window.getSelection().getRangeAt(0)
        }

        if (type === 'focus') {
          this.inputRange = window.getSelection().getRangeAt(0)
        }
      } catch (err) {
        console.log(type)
      }

      return new Promise((resolve, reject) => {
        setTimeout(() => {
          this.inputRange = window.getSelection().getRangeAt(0)
          resolve(this.inputRange)
        }, 0)
      })
    },


    // 延时记录光标到位置
    saveRangeLocal () {
      setTimeout(() => {
        this.inputRange = window.getSelection().getRangeAt(0)
      }, 0)
    },


    // 点击表情，将表情添加到输入框
    sendEmojiIcon (code) {
      let inputNode = document.getElementById('charInput')
      let html = "<img src='"+ this.getIconPic(code) +"' unicode = '" + code + "' alt='' >"
      let sel = window.getSelection()
      let range = this.inputRange
      let el = document.createElement("div")
      let frag = document.createDocumentFragment(), node, lastNode

      if (!inputNode) {
        return
      }

      if (!range) {
        inputNode.focus()
        range = window.getSelection().getRangeAt(0)
      }

      range.deleteContents()
      el.innerHTML = html

      while ((node = el.firstChild)) {
        lastNode = frag.appendChild(node)
      }
      range.insertNode(frag)
      
      if (lastNode) {
        range = range.cloneRange()
        range.setStartAfter(lastNode)
        range.collapse(true)
        sel.removeAllRanges()
        sel.addRange(range)
      }
    },


    // 将输入框中的图片替换为emoji表情
    formatInputCon () {
      let inputValue = document.getElementById('charInput').innerHTML

      inputValue = inputValue.replace(/<img.*?(?:>|\/>)/gi, (val) => {
        let unicode = val.match(/unicode=[\'\"]?([^\'\"]*)[\'\"]?/i)[1]
        let icon = this.emojiIcon
        let iPic = ''

        // 遍历查找Unicode表情
        for (const key in icon) {
          if (icon.hasOwnProperty(key)) {
            const iType = icon[key]
            let flag = false

            for (let index = 0; index < iType.length; index++) {
              const element = iType[index]

              if (element.unicode == unicode) {
                iPic = element.emoji
                flag = true
                break
              }
            }

            if (flag) {break}
          }
        }

        return iPic
      })

      return inputValue
    },


    // 将emoji表情转换为图片
    changeEmojiCon (str) {
      let patt = /[\ud800-\udbff][\udc00-\udfff]/g    // 检测utf16字符正则

      str = str.replace(patt, (char) => {
        let H, L, code
        
        if (char.length === 2) {
          H = char.charCodeAt(0)   // 取出高位
          L = char.charCodeAt(1)   // 取出低位
          code = (H - 0xD800) * 0x400 + 0x10000 + L - 0xDC00   // 转换算法
          return "&#" + code + ";"
        } else {
          return char
        }
      })

      str = str.replace(/&#{1}[0-9]+;{1}/ig, (a) => {
        let unicode = a.replace(/^&#{1}/ig, '')
        
        unicode = unicode.replace(/;{1}$/ig, '')
        unicode = 'U+' + (parseFloat(unicode).toString(16).toUpperCase())

        return "<img src='"+ this.getIconPic(unicode) +"'/>"
      })

      return str
    },


    // 初始化emoji的map对象
    initEmojiPic () {
      let map = new Map()

      for (const key in emojiData.icon) {
        emojiData.icon[key].forEach(item => {
          map.set(item.unicode, require('./../assets/emoji/icon/' + item.unicode + '.png'))
        })
      }

      this.emojiPath = map
    },


    // 通过Unicode码从map中获取emoji
    getIconPic (unicode) {
      return this.emojiPath.get(unicode)
    },

  },


  mounted () {
    this.initEmojiPic()
  }
}
</script>


<style lang="scss">

  .chat_page_con {
    padding: 20px 50px;
    box-sizing: border-box;
    width: 100wv;
    height: 100vh;
    overflow: hidden;
  }

  @font-face {
    font-family: 'iconfont';  /* project id 1098946 */
    src: url('//at.alicdn.com/t/font_1098946_1k69hkxpndp.eot');
    src: url('//at.alicdn.com/t/font_1098946_1k69hkxpndp.eot?#iefix') format('embedded-opentype'),
    url('//at.alicdn.com/t/font_1098946_1k69hkxpndp.woff2') format('woff2'),
    url('//at.alicdn.com/t/font_1098946_1k69hkxpndp.woff') format('woff'),
    url('//at.alicdn.com/t/font_1098946_1k69hkxpndp.ttf') format('truetype'),
    url('//at.alicdn.com/t/font_1098946_1k69hkxpndp.svg#iconfont') format('svg');
  }

  .iconfont {
    font-family:"iconfont" !important;
    font-size:16px;
    font-style:normal;
    -webkit-font-smoothing: antialiased;
    -webkit-text-stroke-width: 0.2px;
    -moz-osx-font-smoothing: grayscale;
  }

  .chatframe-icon {
    display: inline-block;
    height: 40px;
    border-bottom: 1px solid #E8E8E8;
    vertical-align: top;
    margin-left: 10px;

    .iconfont {
      display: inline-block;
      font-size: 20px;
      cursor: pointer;
    }

    .open_emoji_icon {
      color: #FFBF1F;
    }
  }

  .chatframe-input {
    margin-top:10px;
    height: 150px;
  }

  .chatframe_input_con {
    display: inline-block;
    min-width: 500px;
    max-width: 500px;
    min-height: 172px;
    max-height: 172px;
    resize: vertical;
    padding: 5px 15px;
    line-height: 1.5;
    box-sizing: border-box;
    font-size: inherit;
    color: #606266;
    background-color: #fff;
    background-image: none;
    border-radius: 4px;
    transition: border-color .2s cubic-bezier(.645,.045,.355,1);
    border: 1px solid #dcdfe6;
    outline: none;
    overflow-y: scroll;
    margin-bottom: 20px;
    margin-top: 5px;

    img {
      width: 20px;
      height: 20px;
      vertical-align: sub;
    }
  }

  .emoji_tabs_box {
    .el-tabs__nav {
      display: flex;
      width: 100%;
      height: 30px;
    }

    .el-tabs__item {
      flex: 1;
      margin: 0px;
      text-align: center;
      font-weight: 500;
    }

    .emoji_pane_tab {
      font-size: 18px;
    }

    .el-tabs__header {
      margin-bottom: 0;
      padding-top: 5px;
      margin-top: 10px;
      border-top: 1px solid rgba(230, 230, 230, 1);
    }

    .el-tabs__active-bar {
      display: none;
    }
  }

  .el-popover {
    .el-tabs__item {
      height: 25px;
    }

    .el-tabs__active-bar{
      background: #409eff;
    }

    .el-tabs__content {
      height: 140px;
    }

    .el-button+.el-button{
      margin-left: 0;
    }

    .chat_emoji_item {
      width: 25px;
      height: 25px;
      margin: 3px;
      cursor: pointer;
      user-select: none;
    }

    .el-tabs__nav-wrap::after {
      background-color: #fff;
    }
  }

  .scrollbar::-webkit-scrollbar {  
    width: 6px;  /*滚动条宽度*/
  }

  .scrollbar::-webkit-scrollbar-track { 
    border-radius: 6px;  /*滚动条的背景区域的圆角*/
    background-color: #fff;/*滚动条的背景颜色*/  
  }  

  .scrollbar::-webkit-scrollbar-thumb {  
    border-radius: 6px;  /*滚动条的圆角*/
    background-color: #c2c2c2;  /*滚动条的背景颜色*/
  }

  .scrollbar-h::-webkit-scrollbar {  
    height: 6px;  /*滚动条高度*/
  }

  .chatframe-text {
    min-height: 200px;
    width: 500px;
    box-sizing: border-box;
    padding: 10px 15px;
    margin-top: 5px;
    background: #EAEDF1;
    color: #282828;
    border-radius: 4px;
    word-wrap: break-word;
    word-break: break-all;
    line-height: 18px;
    white-space: pre-wrap;
    position: relative;
  }

  .text_emoji {
    img {
      display:inline-block; 
      width: 20px; 
      height: 20px;
      vertical-align: bottom;
    }
  }
</style>

