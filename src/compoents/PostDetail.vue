<template>
  <div id="pd-window">
    <div id="pd-container">
      <div id="pd-close-btn" @click="close()">×</div>
      <div id="pd-content">
        <div id="pd-info" v-bind:style="rStyle">
          <div id="pd-wrap">
            <div id="pd-title">{{post.title}}</div>
            <!--
            <div id="pd-desc"># {{post.milestone && post.milestone.title}} #</div>
            -->
            <span class="post-tags">
              <span class="p-tag" v-for="label in post.labels" v-bind:key="label.id">{{label.name}}</span>
            </span>
            <div class="creat-time p-time">发布于{{new Date(post.created_at).toLocaleDateString().replace(/\//g, '-')}}</div>
            <span>&nbsp;&nbsp;</span>
            <div class="update-time p-time">更新于{{new Date(post.updated_at).toLocaleDateString().replace(/\//g, '-')}}</div>
          </div>
        </div>
        <div id="pd-right">
          <div id="pd-html" v-html="content" @click="clickImg"></div>
          <div id="gh-links">
            在Github中
            <a :href="this.post.html_url" target="_blank" class="gh-show">查看</a>
             / <a :href="commentURL" target="_blank" class="gh-comment">评论</a>
          </div>
          <!--
          <div id="comments">
            <div id="comments-info">
              <span># {{ comments.length }}条评论</span>
              <a :href="commentURL" class="comment-btn">添加评论</a>
            </div>
            <div class="loading-comment" v-if="loading">评论加载中</div>
            <div class="loading-comment" v-if="!loading && comments.length === 0">还没有评论</div>
            <div class="comment" v-for="comment in comments"
              :key="comment.id">
              <div class="comment-left">
                <img :src="comment.user.avatar_url" :alt="comment.user.login" class="comment-avatar">
              </div>
              <div class="comment-right">
                <div class="comment-info">{{ new Date(comment.created_at).toLocaleDateString() }}</div>
                <div class="comment-content">{{ comment.body }}</div>
              </div>
            </div>
          </div>
          -->
          <Vssue :options="vssueOptions" :issueId="post.number"/>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import marked from 'marked'
import { urls } from '../config'

import { VssueComponent } from 'vssue'
import GithubV3 from '@vssue/api-github-v3'
import 'vssue/dist/vssue.css'

export default {
  props: ['post', 'imgs'],
  data () {
    return {
      comments: [],
      oldTitle: '',
      loading: false,
      commentURL: urls.newComment.replace({ number: this.post.number }),
      vssueOptions: {
        api: GithubV3,
        owner: window.username,
        repo: window.reponame,
        clientId: '6f6e1a67c98186406aed',
        clientSecret: '3454ad6273055076753dabf35e7e36361b6b1ea8',
        perPage: 50,
      }
    }
  },
  computed: {
    content () {
      return this.post && this.post.body &&
        this.processMath(marked(this.post.body, {breaks: true}))
    },
    reverse () {
      return this.post.milestone !== null && this.post.milestone.title === '累积'
    },
    rStyle () {
      let label = this.post.labels[Math.floor(Math.random() * this.post.labels.length)]
      return {backgroundImage: `url(${this.imgs[label.name]})`}
    }
  },
  methods: {
    close () {
      this.$emit('closePostWindow')
    },
    processMath (html) {
      // 处理文章中的公式，使用katex渲染
      return html.replace(/\$\$(.*?)\$\$/g, (text, math) => {
        return window.katex.renderToString(math, {
          displayMode: true
        }) // 处理行间公式
      }).replace(/\$(.*?)\$/g, (text, math) => {
        return window.katex.renderToString(math, {
          displayMode: false
        }) // 处理行内公式
      })
    },
    clickImg (evt) {
      const Viewer = window.Viewer
      const config = window.viewerConfig
      if (evt.target.nodeName === 'IMG') {
        const imgViewer = new Viewer(evt.target, config)
        imgViewer.show()
      }
    },
    loadComments () {
      this.loading = true
      fetch(urls.comment.replace({ number: this.post.number })).then(res => res.json())
        .then(res => {
          this.comments = res
          this.loading = false
        })
    }
  },
  created () {
    history.pushState({}, '', `#/post/${this.post.number}`)
    this.oldTitle = document.title
    document.title = this.post.title
    // this.loadComments()
  },
  destroyed () {
    history.pushState({}, '', `#/`)
    document.title = this.oldTitle
  },
  components: {
    'Vssue': VssueComponent,
  },
}
</script>

<style>
@media screen and (max-width: 768px){/* mobile screen */
  #pd-wrap {
    width: 95%;
    box-sizing: border-box;
  }
  #pd-info {
    width: 100%;
  }
  #pd-content {
    flex-wrap: wrap;
    overflow-y: scroll;
  }
  #pd-html {
    padding: 0px 15px;
    line-height: 170%;
    font-size: 1.1em;
  }
  #pd-html pre {
    overflow-x: scroll;
  }
  #pd-container {
    height: 100%;
    width: 100%;
  }
  #comments {
    padding: 20px;
  }
}
@media screen and (min-width: 768px){/* big screen */
  #pd-wrap {
    width: 85%;
  }
  #pd-info {/* similar to mobile */
    width: 100%;
  }
  #pd-content {
    flex-wrap: wrap;
    overflow-y: scroll;
  }
  #pd-container {
    height: 100%;
    width: 75%;
  }
  #pd-html {
    padding: 40px;
    line-height: 170%;
  }
  #pd-html pre {
    width: 100%;
  }
  #comments {
    padding: 40px;
  }
  /*
  #pd-right {
    overflow-y: scroll;
  }
  */
}
#pd-window {
  position: fixed;
  /* background-color: rgba(0, 0, 0, 0.5); */
  height: 100%;
  width: 100%;
  top: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9;
}
#pd-container {
  background-color: #fff;
  box-shadow: 0 50px 100px 100px rgba(0, 0, 0, 0.2);
  position: relative;
}
#pd-close-btn {
  position: absolute;
  right: 0;
  top: 0;
  width: 30px;
  height: 30px;
  font-size: 30px;
  transition: all ease 0.3s;
  margin: 3px;
  cursor: pointer;
  z-index:2;
  text-shadow: 0px 0px 3px #fff;
}
#pd-close-btn:hover {
  transform: scale(1.5);
}

#pd-content {
  display: flex;
  height: 100%;
}
#pd-info {
  background-color: #eee;
  padding: 20px 15px 20px 20px;
  box-sizing: border-box;
  background-size: cover;
  background-position: center;
}
#pd-title {
  font-size: 26px;
  margin-bottom: 10px;
}
#pd-wrap {
  background-color: rgba(255, 255, 255, 0.8);
  padding: 20px;
}
#pd-html {
  width: 100%;
  box-sizing: border-box;
  color: #444;
  border-bottom: 2px solid rgba(222, 222, 222, 0.5);
}
#pd-html img {
  width: 100%;
  cursor: pointer;
}
#pd-html blockquote {
  border-left: 5px solid #eee;
  -webkit-margin-start: 0;
  padding-left: 10px;
}
#pd-html a {
  color: #67aeeb;
}
#pd-html a img {
  width: auto;
  max-width: 100%;
}

#pd-html pre {
  background: #f8f8f8;
  font-size: 1rem;
  padding: .67rem 1.3rem;
  margin: 0;
  box-sizing: border-box;
}
#pd-html pre code {
  color: #666;
}
#pd-html h1:before,
#pd-html h2:before {
  margin-right: 5px;
}
#pd-html h1,
#pd-html h2 {
  font-weight: normal;
}
#pd-html ol,
#pd-html ul {
  -webkit-padding-start: 20px;
}

#pd-right {
  height: 100%;
  width: 100%;
}

#comments {
  border-top: 1px solid #eee;
}

#comments-info {
  margin-bottom: 20px;
  font-size: 14px;
  display: flex;
  justify-content: space-between;
}

.no-comment {
  text-align: center;
}

.comment {
  display: flex;
  border-bottom: 1px #eee dotted;
  margin-bottom: 10px;
  padding-bottom: 10px;
}

.comment:last-child {
  border: 0;
}

.comment-left {
  width: 40px;
}

.comment-avatar {
  height: 40px;
  width: 40px;
  border: 1px solid #eee;
  border-radius: 5px;
}

.comment-right {
  padding-left: 15px;
  width: 100%;
}

.comment-info {
  font-size: 14px;
  color: #aaa;
  line-height: 20px;
}

.comment-btn {
  color: dodgerblue;
}

.loading-comment {
  text-align: center;
  font-size: 14px;
  color: #aaa;
}

.p-tag {
  border-radius: 10px;
  padding: 0px 10px;
  margin-right: 5px;
  display: inline-block;
  background-color: #eee;
  margin-bottom: 5px;
  font-size: 0.8em
}

.p-time {
  display: inline-block;
  font-size: 0.85em
}

#gh-links {
  text-align: right;
  width: 100%;
  padding: 10px 13px 13px;
  box-sizing: border-box;
  border-bottom: 2px solid rgba(211, 211, 211, 0.5);
  color: #2c3e50
}

#gh-links > a {
  color: rgb(62,175,124)
}

.vssue {
  padding: 13px
}

.vssue-new-comment {
  display: none;
}
/*
login doesn't work, hide the element
*/

</style>
