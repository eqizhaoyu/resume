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
        interval: 5,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* Inspired by http://strml.net/
* 源码地址 https://github.com/eqizhaoyu/resume.git
* 大家好，我是戚兆宇。
* 我来写一份简历！
*/

/* 给所有元素加上过渡效果 */
* {
  transition: all .2s;
}
/* 设置背景颜色 */
html {
  color: rgb(222,222,222);
  background: rgb(0,43,54);
}
/* 设置边框 */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  overflow: auto;
  width: 90vw;
  margin: 2.5vh 5vw;
  height: 90vh;
}
/* 太高了 */
.styleEditor {
  height: 45vh;
}
/* 代码高亮 */
.token.selector{
  color: rgb(133,153,0);
}
.token.property{
  color: rgb(187,137,0);
}
.token.punctuation{
  color: yellow;
}
.token.function{
  color: rgb(42,161,152);
}

/* 加3D效果 */
html{
  perspective: 1000px;
}
.styleEditor {
  position: fixed; left: 0; top: 0;
  transform: rotateX(-10deg) translateZ(-50px) ;
}

/* 准备一个编辑器 */
.resumeEditor{
  position: fixed;
  top: 50%; left: 0;
  padding: .5em;  margin: 2.5vh;
  width: 95vw; height: 45vh;
  border: 1px solid;
  background: white; color: #222;
  overflow: auto;
}
/* 开始写简历 */


`,
          `
/*将Markdown格式翻译成HTML
 *再对HTML加点样式
*/
`, `
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
.resumeEditor ol li::before {
  counter-increment: section;
  content: counters(section, ".") " ";
  margin-right: .5em;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
`],
        currentMarkdown: '',
        fullMarkdown: `戚兆宇
====
坐标：天津市。

求职岗位：
----
  - Java软件工程师

个人简介：
----
  - 学历：本科
  - 年龄：26
  - 工作经验：3年
  - 政治面貌：中共党员
  - 电话：18222705770
  - 邮箱：qizhaoyuqq@163.com

教育经历：
----
  - 天津理工大学 学士学位

数据库设计
----
  - 结构化数据库设计
  - noSQL设计

后端开发
----
  - 用户管理
  - 单点登录
  - 第三方登录
  - 权限管理
  - 交易系统
  - 支付系统
  - 社区系统
  - 博客系统
  - 公众号开发
  - 小程序开发
  - API接口

前端开发
----
  - Web前端开发
  - 移动终端(Native App, Hybrid App)

技术及语言
----
  - Java: SpringMVC, SpringCloud, Hibernate, iBatis, spark, sql2o, HikariCP, freemarker, okHttp, retrofit, RxJava
  - Kotlin: ktor, exposed, anko
  - Node.js: express, angular, ionic, react, cordova, meteor, electron, axios
  - Swift: Vapor, ReactiveSwift
  - Golang: hugo, beego, gorm, sqlx, matcha
  - Python: tushare, pandas, numpy, matplotlib
  - DotNet and PHP
  - DB: SQLServer, Oracle, MySQL/MariaDB, MongoDB, graphQL, redis, memcached
  - WebServer: apache, nginx, tomcat, netty, jetty
  - OS: Ubuntu, CentOS, MacOS, Windows
  - Others: Docker, git, Xmind，Axure

工作经历
====

1. 大宇宙信息创造（中国）有限公司
2. 朗威科技有限公司
3. 天地图（天津）有限公司
4. 有谦软联科技有限公司

链接
----

* [GitHub](https://github.com/eqizhaoyu)
* [技术博客](qizhaoyu.gitlab.io)
* [微信公众号](Q氏先生)

勾引方式
----

* 微信：qhj720

小结
----
* 本人java开发，菜鸡一枚。偶然从http://strml.net/
* 看到炫酷简历，决定也搞一份玩玩。
* 总结一下个人知识库:
* 掌握Java语言，拥有扎实的Java编程功底和良好的编码习惯
* 熟练使用Eclipse 和IDEA等编程工具，熟悉MAVEN、GIT和SVN
* 熟练使用SpringMVC和MyBatis框架，了解Struts2、Hibernate和SpringBoot框架
* 熟悉MySQL数据库和熟练撰写SQL语句及存储过程及数据库优化，了解Oracle数据库
* 了解HTML，CSS，JavaScript和Ajax调用数据，easyUI、vue、bootStrap等前端框架
* 熟悉Linux系统及基本操作命令，能够创建centos/ubuntu虚拟机并在上面部署tomcat、 mysql等服务器环境及搭建部署conflunce、wiki、jira、禅道等办公软件
* 了解Redis、zookeeper、kafka、hadoop和hbase分布式和负载均衡的相关知识

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
          this.$nextTick(() => {
            this.$refs.resumeEditor.goTop()
          })
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
    min-height: 100vh; position: relative;
  }

  html {
    min-height: 100vh;
  }
  *{
    box-sizing: border-box;
  }

</style>
