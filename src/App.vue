<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
  import StyleEditor from './components/styleEditor'
  import ResumeEditor from './components/resumeEditor'
  import './assets/reset.css'

  export default {
    name: 'app',
    components: {
      StyleEditor,
      ResumeEditor
    },
    data() {
      return {
        interval: 30,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* 你好，我叫董鑫强
* 马上要找实习了，得抓紧写一份简历呀。
* 说做就做！
*/

/* 首先给所有元素加上过渡效果 */
* {
  -webkit-transition: all .3s;
  transition: all .3s;
}
/* 白色背景太单调了，我们来点背景 */
html {
  color: rgb(222,222,222);
  background: rgb(0,43,54);
}
/* 文字离边框太近了 */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  margin: .5em;
  overflow: auto;
  width: 45vw;
  height: 90vh;
}
/* 代码高亮 ( 使用 prismjs 实现 )*/
.token.selector{ color: rgb(133,153,0); }
.token.property{ color: rgb(187,137,0); }
.token.function{ color: rgb(42,161,152); }

/* 加点 3D 效果呗 */
html{
  -webkit-perspective: 1000px;
  perspective: 1000px;
}
.styleEditor {
  position: fixed;
  left: 0;
  top: 0;
  -webkit-transition: none;
  transition: none;
  -webkit-transform: rotateY(10deg) translateZ(-100px) ;
  transform: rotateY(10deg) translateZ(-100px) ;
}

/* 接下来准备一个编辑器 */
.resumeEditor{
  position: fixed;
  right: 0;
  top: 0;
  padding: .5em;
  margin: .5em;
  width: 48vw;
  height: 90vh;
  border: 1px solid;
  background: white;
  color: #222;
  overflow: auto;
}
/* 好了，开始写简历了 */


`,
          `
/* 你可能发现这是 Markdown 格式的，看起来好麻烦
 * 超级变变变 (使用 marked 实现)
 */
`
          ,
          `
/* 再来点样式 */
.resumeEditor{
  padding: 2em;
}
.resumeEditor p,.resumeEditor li{
  line-height: 1.5;
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
  font-size: 30px;
  margin: 1em 0 .5em;
}
.resumeEditor ul,.resumeEditor ol{
  list-style: none;
}
.resumeEditor ul> li::before{
  content: '•';
  margin-right: .5em;
}
.resumeEditor ol {
  counter-reset: section;
}
.resumeEditor ol li::before {
  counter-increment: section;
  content: counters(section, "") " ";
  margin-right: .5em;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
/* 完成啦 */
/* cooooooooooooooooool */
`],
        currentMarkdown: '',
        fullMarkdown: `## 董鑫强
----

资浅前端工程师，现在西安读书，前端真的很有趣！

## 技能
----

* 熟悉HTML(5)，CSS(3)，JavaScript；了解ES6规范及特性；
* 熟悉Bootstrap，jQuery；熟悉Sass，Compass；熟悉Vue.js，了解React；
* 熟悉PHP和MySQL，有过简单开发经验。

## 经历
----

1. 长安大学网络工程系
2. ？？？

## 链接
----

* [我的博客](http://daryldong.com/)
* [GitHub](https://github.com/DarylsJoy)

> 这个效果咋来的？点这里： [我是这里](https://github.com/DarylsJoy/animate-resume/)

`
      }
    },
    created() {
      this.makeResume()
    },

    methods: {
      // 异步处理左右两个区域交替输入文字
      makeResume: async function () {
        await this.progressivelyShowStyle(0)
        await this.progressivelyShowResume()
        await this.progressivelyShowStyle(1)
        await this.showHtml()
        await this.progressivelyShowStyle(2)
      },
      showHtml: function () {
        return new Promise((resolve, reject) => {
          this.enableHtml = true
          resolve()
        })
      },
      progressivelyShowStyle(n) {
        return new Promise((resolve, reject) => {
          let interval = this.interval
          let showStyle = (async function () {
            let style = this.fullStyle[n]
            if (!style) {
              return
            }
            // 计算前 n 个 style 的字符总数
            let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0)
            let prefixLength = length - style.length
            if (this.currentStyle.length < length) {
              let l = this.currentStyle.length - prefixLength
              let char = style.substring(l, l + 1) || ' '
              this.currentStyle += char
              if (style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
                this.$nextTick(() => {
                  this.$refs.styleEditor.goBottom()
                })
              }
              setTimeout(showStyle, interval)
            } else {
              resolve()
            }
          }).bind(this)
          showStyle()
        })
      },
      progressivelyShowResume() {
        return new Promise((resolve, reject) => {
          let length = this.fullMarkdown.length
          let interval = this.interval
          let showResume = () => {
            if (this.currentMarkdown.length < length) {
              this.currentMarkdown = this.fullMarkdown.substring(0, this.currentMarkdown.length + 1)
              let lastChar = this.currentMarkdown[this.currentMarkdown.length - 1]
              let prevChar = this.currentMarkdown[this.currentMarkdown.length - 2]
              console.log(prevChar)
              if (prevChar === '\n' && this.$refs.resumeEditor) {
                this.$nextTick(() => this.$refs.resumeEditor.goBottom())
              }
              setTimeout(showResume, interval)
            } else {
              resolve()
            }
          }
          showResume()
        })
      }
    }
  }

</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
  html {
    min-height: 100vh;
  }
  * {
    -webkit-transition: all .3s;
    transition: all .3s;
  }
</style>
