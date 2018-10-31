<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
  import StyleEditor from './components/StyleEditor'
  import ResumeEditor from './components/ResumeEditor'
  import './assets/reset.css'

  export default {
    name: 'app',
    components: {
      StyleEditor,
      ResumeEditor
    },
    data() {
      return {
        interval: 40,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* 身为一个没有文艺细胞的程序员
* 我只能用代码给你们展示广告作品
* 希望大家能够喜欢
*/

/* 首先给所有元素加上过渡效果 */
* {
  transition: all .3s;
}
/* 白色背景太单调了，我们来点背景 */
html {
  color: rgb(222,222,222); background: rgb(0,43,54);
}
/* 文字离边框太近了 */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  margin: .5em;
  overflow: auto;
  width: 45vw; height: 90vh;
}
/* 代码高亮 */
.token.selector{ color: rgb(133,153,0); }
.token.property{ color: rgb(187,137,0); }
.token.punctuation{ color: yellow; }
.token.function{ color: rgb(42,161,152); }

/* 加点 3D 效果呗 */
html{
  perspective: 1000px;
}
.styleEditor {
  position: fixed; left: 0; top: 0;
  -webkit-transition: none;
  transition: none;
  -webkit-transform: rotateY(10deg) translateZ(-100px) ;
          transform: rotateY(10deg) translateZ(-100px) ;
}

/* 接下来我给自己准备一个编辑器 */
.resumeEditor{
  position: fixed; right: 0; top: 0;
  padding: .5em;  margin: .5em;
  width: 48vw; height: 90vh;
  border: 1px solid;
  background: white; color: #222;
  overflow: auto;
}
/* 好了，我开始做广告了 */


`,
          `
/* 这个广告好像差点什么
 * 对了，这是 MarkDown 格式的，我需要变成对用户更友好的格式
 * 简单，用开源工具翻译成 HTML 就行了
 */
`
          ,
          `
/* 再对 HTML 加点样式 */
.resumeEditor{
  padding: 2em;
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
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
.resumeEditor ul{
	list-style: none;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
`],
        currentMarkdown: '',
        fullMarkdown: `<h1 align="center">太吾绘卷The Scroll Of Taiwu</h1><p align="center"><img src="https://yiyun-1253940215.cos.ap-shanghai.myqcloud.com/20181013203212.png" class="full-image"/></p>




简介
----
《太吾绘卷》是一款以神话和武侠为题材的独立游戏

* 扮演神秘的“太吾氏传人”，生活在以古代中华神州为背景的架空世界中
* 拜访世界各地的武林门派，学习种类繁多的功法绝艺
* 与人义结金兰，或结下血海深仇
* 兴建自己的村庄，经营各种产业
* 与自己的挚爱生儿育女，缘定三生
* 直到你通过一代又一代传人的努力和牺牲，面对太吾氏的宿敌，决定世界的命运!

未知的江湖
----

* 完全随机生成的地图、完全随机生成的NPC、敌人
* 你的每次游戏都会是一次崭新的冒险。
* 你能够从十五个风格迥异的武林门派中，学习到上千种功法绝艺
* 从几乎任何NPC身上，学习到上百种中华传统技艺。


相关故事
----

1. 1、代码谣言             
	>* “作者在游戏里写了几十万个IF语句”
	>* “整个游戏只有一个main”
	>* “所有数据表逻辑都在一个列里，有1W多行……”

2. 2、团队谣言
	 >* 3.5人的小团队，没有什么游戏制作的经验
	 >* 凭借着一腔对游戏的热爱，投身游戏制作的事业当中去
	 >* 中间还有程序员跑路，主创人员自学编程从头开始

营销
----

 * 实现国产游戏在 Steam 上的惊天逆转，首周 30W 份，实现畅销榜第一。

链接
----

* [Steam](https://store.steampowered.com/app/838350/_The_Scroll_Of_Taiwu/?l=schinese)
* [Animation Advertisement](https://github.com/yi-yun/)

`
      }
    },
    created() {
      this.makeResume()
    },

    methods: {
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
            if (!style) { return }
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
  *{
    box-sizing: border-box;
  }
</style>
