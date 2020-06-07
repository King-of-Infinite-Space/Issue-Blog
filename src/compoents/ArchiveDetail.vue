<template>
  <div id="archive-window">
    <div id="archive-container">
      <div id="archive-close-btn" @click="close()">×</div>
      <div id="ar-content" v-if="archive">
        <div id="ar-info" v-bind:style="style(archive.name)">
          <div id="ar-wrap">
            <div id="ar-title">{{archive.name}}</div>
            <!--
            <div id="ar-desc">{{archive.description}}</div>
            -->
          </div>
        </div>
        <div id="ar-list">
          <Post :posts="posts" :loading="loading" noMore=true
            v-on:readPost="read"/>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import {urls} from '../config'
import Post from './Post.vue'
export default {
  props: ['archive', 'imgs'],
  data () {
    return {
      posts: [],
      loading: false,
    }
  },
  created () {
    this.loading = true
    fetch(urls.issue.query({ labels: this.archive.name })).then(res => res.json()).then(res => {
      this.posts = res
      this.loading = false
    })
  },
  methods: {
    close () {
      this.$emit('closeArchiveWindow')
    },
    read (post) {
      this.$emit('readPost', post)
    },
    style (name) {
      // return {backgroundImage: `url('../imgs/${name}.jpg')`}
      return {backgroundImage: `url(${this.imgs[name]})`}
    },
  },
  components: {
    Post
  }
}
</script>

<style>

@media screen and (max-width: 768px){
  #ar-info {
    height: 20%;
    width: 100%;
  }
  #ar-content {
    flex-wrap: wrap;
  }
  #archive-container {
    height: 100%;
    width: 100%;
  }
  #ar-list {
    height: 80%;
  }
}
@media screen and (min-width: 768px){
  #ar-info {
    height: 100%;
    min-width: 20%;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  #archive-container {
    height: 100%;
    width: 700px;
  }
  #ar-list {
    height: 100%;
  }
}
#archive-window {
  position: fixed;
  /* background-color: rgba(0, 0, 0, 0.5); */
  height: 100%;
  width: 100%;
  top: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}
#archive-container {
  background-color: #fff;
  box-shadow: 0 50px 100px 100px rgba(0, 0, 0, 0.2);
  position: relative;
}
#archive-close-btn {
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
}
#archive-close-btn:hover {
  transform: scale(1.5);
}

#ar-content {
  display: flex;
  height: 100%;
}
#ar-info {
  background-color: #eee;
  padding: 20px 15px;
  box-sizing: border-box;
  background-size: cover;
  background-position: center;
}
#ar-title {
  font-size: 1.5rem;
  margin-bottom: 10px;
}
#ar-list {
  width: 100%;
  overflow-y: scroll;
}
#ar-wrap {
  background-color: rgba(255, 255, 255, 0.8);
  padding: 20px;
  width: 50%;
}

</style>
