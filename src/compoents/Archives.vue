<template>
  <div id="milestones">
    <div class="loading" v-if="milestones.length == 0"></div>
    <div class="ms-wrap" v-else v-bind:class="{'msw-hidden': hidden}">
      <div class="milestone" v-for="(ms, index) in milestones" v-bind:class="{'ms-hidden': hidden}" v-bind:key="ms.id" @click="msClick(index)" v-bind:style="style(ms.name)">
        <div class="ms-mask">
          <div class="ms-title">{{ms.name}}</div>
          <!--
          <div class="ms-desc">{{ms.open_issues}}篇文章</div>
          -->
        </div>
      </div>
    </div>
  </div>
</template>
<script>

export default {
  props: ['milestones', 'imgs'],
  data () {
    return {
      hidden: false
    }
  },
  methods: {
    style (name) {
      // return {backgroundImage: `url('/imgs/${name}.jpg')`}
      return {backgroundImage: `url(${this.imgs[name]})`}
    },
    msClick (index) {
      this.$emit('openArchiveWindow', index)
    }
  }
}
</script>
<style>
  @media screen and (max-width: 768px){
    .milestone {
      width: 100px;
      min-height: 40px;
      min-width: 100px;
      margin-right: 5px;
    }
    .ms-wrap {
      flex-wrap: nowrap;
      overflow-x: scroll;
      width: 100%;
      padding: 5px 0;
    }
  }
  @media screen and (min-width: 768px){
    .milestone {
      width: 100%;
      max-width: 120px;
      height: 43px;
      margin-bottom: 5px;
      transition: transform 0.2s;
    }
      .milestone:hover {
        transform: translate(15px,0);
    }
    .ms-wrap {
      flex-direction: column;
    }
  }
  .ms-wrap {
    display: flex;
  }

  .milestone {
    background-size: cover;
    background-position: center;
    box-sizing: border-box;
    cursor: pointer;
    box-shadow: 0 0 30px rgba(0, 0, 0, 0.1);
    background-color: rgba(0, 0, 0, 0.3);
  }

  .ms-mask {
    background-color: rgba(0, 0, 0, 0.1);
    height: 100%;
    display: flex;
    justify-content: center; /* align horizontal */
    align-items: center; /* align vertical */
  }
  .ms-title {
    font-size: 1rem;
    text-align: center;
    color: white;
    /*
    border-bottom: 1px dotted rgba(255, 255, 255, 0.5);
    line-height: 20px;
    margin: 0 20px;
    padding: 15px 0 5px 0;
    */
  }
  .ms-desc {
    font-size: 12px;
    text-align: center;
    color: #fff;
    padding: 5px 0 10px 0;
  }
</style>
