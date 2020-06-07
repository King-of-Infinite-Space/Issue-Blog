<template>
  <div>
    <div id="nav">
      <div class="layout" id="links">
        <a :href="link[1]" v-for="(link, index) in links" :key="index">
          {{link[0]}}
        </a>
      </div>
    </div>
    <div id="banner">
      <div id="motto">{{ motto }}</div>
    </div>
    <div class="layout" id="main-container">
      <div id="archives">
        <Archives :milestones="archives" :imgs="imgs" v-on:openArchiveWindow="openArchiveWindow"/>
        <!--
        <div class="ls-hidden hot-articles">
          <Hot :hotPosts="hotPosts" v-on:readPost="readPost"/>
        </div>
        -->
      </div>
      <div id="post-container">
        <Post :posts="posts" :loading="loadingPost" :noMore="noMore"
          v-on:readPost="readPost"
          v-on:loadMorePosts="loadPosts"/>
      </div>
      <!--
      <div class="xs-hidden hot-articles">
        <Hot :hotPosts="hotPosts" v-on:readPost="readPost"/>
      </div>
      -->
      <div id="return-top" @click="returnTop"></div>
    </div>
    <transition name="popupPage">
      <PostDetail v-if="showPost" :post="showingPost" :imgs="imgs" v-on:closePostWindow="closePostWindow"/>
    </transition>
    <transition name="popupPage">
      <ArchiveDetail v-if="showArchive" :archive="showingArchive" :imgs="imgs"
      v-on:closeArchiveWindow="closeArchiveWindow"
      v-on:readPost="readPost"/>
    </transition>
    <div id="footer">
      <div class="layout" id="footer-content">
        <div id="repo-link">
          <a href="https://github.com/King-of-Infinite-Space/thoughts/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc" target="_blank">View in Github</a>
        </div>
        <div class="footer-info">
          Powered by <a href="https://github.com/Yidadaa/issue-blog" target="_blank">issue-blog</a>
        </div>
        <div class="footer-links">
          <!--
          <span>友情链接:</span>
          <a :href="link[1]" v-for="(link, index) in friendLinks" :key="index">{{link[0]}}</a>
          -->
        </div>
      </div>
      <div id="footer-wrap"></div>
    </div>
  </div>

</template>
<script>
import {urls} from './config'
import Post from './compoents/Post.vue'
import Archives from './compoents/Archives.vue'
import Hot from './compoents/Hot.vue'
import ArchiveDetail from './compoents/ArchiveDetail.vue'
import PostDetail from './compoents/PostDetail.vue'

const preventWindowScroll = () => {
  // 阻止弹窗弹出时，窗口滚动
  // const [posx, posy] = [window.scrollX, window.scrollY]
  // window.onscroll = () => window.scrollTo(posx, posy)
  const before = document.body.clientWidth
  document.body.style.overflow = 'hidden'
  const after = document.body.clientWidth
  document.body.style.paddingRight = `${after - before}px`
}

const allowWindowScroll = () => {
  // 允许窗口滚动
  // window.onscroll = null
  document.body.style.overflow = ''
  document.body.style.paddingRight = `0px`
}

window.onscroll = () => {
  if (document.body.getBoundingClientRect().top < -1000) {
    document.getElementById('return-top').className = 'show-top-btn'
  } else {
    document.getElementById('return-top').className = ''
  }
}

export default {
  name: 'blog',
  data () {
    return {
      posts: [], // 文章列表
      loadingPost: false, // 是否在加载文章
      curPage: 1, // 当前页
      noMore: false, // 没有更多了
      archives: [], // 分类
      hotPosts: [], // 热门文章

      showArchive: false, // 是否打开分类窗口
      showArchiveIndex: -1,

      showPost: false, // 查看文章内容
      showingPost: {}, // 文章详情
      imgs: { // 图床 sm.ms, imgurl.org
        '赛博空间': 'https://i.loli.net/2019/07/24/5d380588cec5c77214.jpg',
        '绿茵故事': 'https://i.loli.net/2019/07/24/5d380f266eb3322085.jpg',
        '时光碎片': 'https://i.loli.net/2019/07/24/5d3834147841879201.jpg',
        '撷叶拾英': 'https://i.loli.net/2019/07/26/5d3ae6116360c39035.jpg',
        '言不及义': 'https://i.loli.net/2019/07/26/5d3aec2a667dc93355.jpg',
        '好行小慧': 'https://i.loli.net/2019/07/26/5d3af278f122a30361.jpg',
        '有文无类': 'https://i.loli.net/2019/07/26/5d3aec2ac857124936.jpg',
        '外置记忆': 'https://b2.bmp.ovh/imgs/2019/07/203eac5466049979.jpg',
        '雕虫小技': 'https://i.loli.net/2019/07/26/5d3af8a4806a030021.jpg',
        '陋词拙句': 'https://i.loli.net/2019/07/28/5d3d79b74671448737.jpg',
        '今是何世': 'https://b2.bmp.ovh/imgs/2019/07/2ff2c89d17dda061.jpg'
      }
    }
  },
  computed: {
    showingArchive () {
      return this.archives[this.showArchiveIndex]
    },
    links () {
      return window.links
    },
    friendLinks () {
      return window.friendLinks
    },
    motto () {
      return window.motto
    }
  },
  methods: {
    loadPosts () {
      const url = urls.issue
      if (this.noMore) return // 没有更多内容了

      this.loadingPost = true
      fetch(url.query({ page: this.curPage, sort: 'updated' })).then(res => res.json()).then(res => {
        const posts = res.map(post => {
          const reg = /!\[.*\]\((.*)\)/
          const match = post.body.match(reg) // 找出文中第一张图
          const sliceNum = 150
          const cnchars = (post.body.slice(0, sliceNum).match(/[^\x00-\xff]/g) || '').length
          // console.log(cnchars)
          return Object.assign({}, post, {
            short: post.body.slice(0, sliceNum - parseInt(cnchars / 2))
              .replace(/[*#\-`>]/g, ' ') + '...',
            image: match && match[1]
          })
        })
        if (res.length > 0) {
          this.curPage += 1
        } else {
          this.noMore = true
        }
        this.posts = [].concat(this.posts, posts)
        this.loadingPost = false
      })
    },
    loadArchives () {
      const url = urls.milestones.url
      if (!url) return
      fetch(url).then(res => res.json()).then(res => {
        this.archives = res
      })
    },
    loadHotPosts () {
      const url = urls.issue
      fetch(url.query({ sort: 'comments' })).then(res => res.json()).then(res => {
        const hotPosts = res.filter(post => post.comments > -1) // 过滤掉没有评论的
        this.hotPosts = hotPosts.length > 5 ? hotPosts.slice(0, 5) : hotPosts // 只取前五个
      })
    },
    readPost (post) {
      this.showingPost = post
      this.openPostWindow()
    },
    closeArchiveWindow () {
      allowWindowScroll()
      this.showArchive = false
      this.showArchiveIndex = -1
    },
    openArchiveWindow (index) {
      preventWindowScroll()
      this.showArchive = true
      this.showArchiveIndex = index
    },
    closePostWindow () {
      allowWindowScroll()
      this.showPost = false
      this.showingPost = {}
    },
    openPostWindow () {
      preventWindowScroll()
      this.showPost = true
    },
    onLoaded () {
      // 数据加载完成后，如果url有变，跳转到指定post
      const hash = location.hash.replace('#/', '')
      if (!hash) return
      const hashInfo = hash.split('/')
      /*
      let re = /\?code=\w+/
      let match = location.href.match(re)
      if (match) {
        // location.href = location.href.replace(re, '') + match[0]
        location.hash += match[0]
      }
      */
      if (hashInfo[0] === 'post') {
        this.loadSinglePost(hashInfo[1]).then(res => res.title && this.readPost(res))
      }
    },
    loadSinglePost (number) {
      // 加载指定post
      let token = window.authToken
      let url = `${urls.issue.url}/${number}`
      if (token) return fetch(url).then(res => res.json())
      else {
        console.log('authorized call')
        return fetch(url, {headers: {'Authorization': 'token ' + token}}).then(res => res.json())
      }
    },
    returnTop () {
      window.scrollTo(0, 0)
    }
  },
  created () {
    this.onLoaded() // 判断是否需要加载指定post
    this.loadPosts(0)
    this.loadArchives()
    // this.loadHotPosts()
  },
  components: {
    Post, Archives, Hot, ArchiveDetail, PostDetail
  }
}
</script>
<style>
@import './style/animation.css';
@import './style/global.css';

@media screen and (max-width: 768px){
  /*手机屏幕 <768px*/
  .layout {
    width: 100%;
  }
  #links {
    padding: 0 20px;
  }
  #banner {
    height: 100px;
  }
  #motto {
    font-size: 2rem;
    font-weight: lighter;
  }
  #archives {
    width: 100%;
    box-sizing: border-box;
  }
  #main-container {
    flex-wrap: wrap;
    position: relative;
  }
  #post-container {
    width: 100%;
  }
  .ls-hidden {
    display: none;
  }
}
@media screen and (min-width: 768px) and (max-width: 1200px){
  /*平板屏幕 >768px <992x*/
  .layout {
    width: 80%!important;
  }

}
@media screen and (min-width: 768px){
  /*大屏幕 >992px*/
  .layout {
    width: 65%;
  }
  #banner {
    height: 120px;;
  }
  #motto {
    font-size: 3rem;
  }
  #archives {
    /*min-width: 300px;
    width: 30%;*/
    padding: 10px;
    width: 40%;;
  }
  #main-container {
    flex-direction: row-reverse;
    flex-wrap: nowrap;
  }
  #post-container {
    width: 70%;
    padding-top: 10px;
  }
  .xs-hidden {
    display: none;
  }
}

#nav {
  position: absolute;
  width: 100%;
  height: 40px;
  background-color: rgba(0, 0, 0, 0.2);
  display: none;
}
#links {
  line-height: 40px;
  box-sizing: border-box;
  padding: 0 15px;
}
#links a {
  color: #eee;
  font-size: 16px;
  position: relative;
  padding-bottom: 5px;
  margin-right: 10px;
  text-shadow: 1px 1px 0 rgba(0, 0, 0, 0.3);
  text-decoration: none;
}

#links a:after {
  content: '';
  width: 100%;
  height: 3px;
  background-color: #eee;
  display: inline-block;
  position: absolute;
  bottom: 0;
  left: 0;
  transform: translateY(10px);
  opacity: 0;
  transition: all ease 0.5s;
}

#links a:hover:after {
  transform: translateY(0px);
  opacity: 1;
}

.layout {
  margin: auto;
}

#banner {
  background-image: url("https://i.loli.net/2019/07/30/5d401851275c882243.jpg");
  background-size: cover;
  background-position: center;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 0 50px rgba(0, 0, 0, 0.2);
}

#motto {
  color: #fff;
  text-shadow: 0 5px 5px rgba(0, 0, 0, 0.5);
}

#archives {
  height: auto;
}

#main-container {
  display: flex;
}

#post-container {
  height: auto;
}

.hot-articles {
  width: 100%;
}

#footer {
  position: relative;
  overflow: hidden;
  box-sizing: border-box;
  max-height: 60px;
}

#footer-wrap {
  height: 100%;
  width: 100%;
  position: absolute;
  top: 0;
  z-index: -1;
  box-sizing: border-box;
  /*
  filter: blur(20px) brightness(70%);*/
  background-image: url("https://i.loli.net/2019/07/30/5d401851275c882243.jpg");
  background-size: cover;
  background-position-y: bottom;
  background-position-x: center;
}

#footer-content {
  display: flex;
  color: #fff;
  text-shadow: 0 0 1px rgba(0, 0, 0, 0.2);
  padding: 20px;
  line-height: 1.5;
  justify-content: space-between;
}
.footer-links {
  margin-left: 10px;
}
.footer-info a {
  color: #fff;
}
.footer-links a {
  color: #fff;
  margin-right: 5px;
}
#repo-link a {
  color: #fff;
}
#return-top {
  position: fixed;
  bottom: 10px;
  right: 10px;
  height: 50px;
  width: 50px;
  background-color: #fff;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
  border-radius: 5px;
  background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAYAAACqaXHeAAACzElEQVR4Xu2b7XHUMBCGX1UAHUAHQAXJXgMkFZBUQKgA6CB0kBJCA6ekgkAHoYN0oMzOyMZxLFsfq/WMLf3y+HQ6vY9WK61WZ7DzYjr91tq3AL4DuADAz1ssjwBuiOhnJ24I4BrA1y2qntB0TkS3/L4HcDwe74wxJzsB8IuIrl4AsNYykc87AfCNiNji/1uAtfa9c46t4N2WITjn7o0xZ0T09ALAwBmebhjAExH9GerrfcCGRc9KawD2OvKv9gF7BbHaFOBVB8AXD/732DlpDcgqAKy1HwHY0Zb7kohutISvNgUC4rv+qENQtYAF8atAUAMQKV4dggqARPGqEKoDyBSvBqEqgELxKhCqARASXx1CFQBL4p1zf40xH4ZrvnPuH+8LjDFvAnuBKkukOIAl8QAuAfAukM8f++Lj9Ct/JqEGQRRAjHje7Vlrf0wBOBwOp9yGJgQxALHiecjnAPjP1SCIAEgRHwNAE0IxgFTxsQC0IBQByBGfAkADQjaAXPGpAGpDyAJQIj4HQE0IyQBKxecCqAUhCYCE+BIANSBEA5ASXwpAGkIUAEnxEgAkISwCkBYvBUAKwiyAGuIlAUhACALwN0YefOQ2FaFmh6dLsUAgHA6+XgigOAv8iYj4dsirMgeAs8R8di8qXtoCus4tQOjvA4zFzAHg5AVbwLhkj/ygs8FwOHX0h/VnIPRXYqIB+JHiTE2XvuJXxeJrWUDIEvighc8ZQmBjVwFu4DY0j1JHTdoHjH/f+68zAI9EdDfXv0UAqeJi6tcGENOHrk4DkEJLqm6zgJlDUSnIse20KRBLSrJemwJtCoQTI5KWFtNW8wExlKTrNB/QfEDzAVXC4Zyp2pxgDrXS7zQn2Jxgc4LNCYbuCJX6l9Tvt1UglZhE/bYKTKwCAPhfI3ySq1rWmgJTWSeRnEMqvVUA+OTIhXOOVwO+HntNRPysXp4BhQajX7NS3goAAAAASUVORK5CYII=);
  background-size: 30px 30px;
  background-repeat: no-repeat;
  background-position: center center;
  cursor: pointer;
  transition: transform ease .3s;
  transform: translateY(200px);
  z-index: 9999;
}

.show-top-btn {
  transform: translateY(0)!important;
}

.popupPage-enter-active {
  animation: window-in 0.4s ease-in-out;
}

.popupPage-leave-active {
  animation: window-in 0.4s ease-in-out reverse;
}
</style>
