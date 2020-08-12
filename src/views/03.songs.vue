<template>
  <div class="songs-container">
    <div class="tab-bar">
      <span class="item" @click="tag=0" :class="{active:tag==0}">全部</span>
      <span class="item" @click="tag=7" :class="{active:tag==7}">华语</span>
      <span class="item" @click="tag=96" :class="{active:tag==96}">欧美</span>
      <span class="item" @click="tag=8" :class="{active:tag==8}">日本</span>
      <span class="item" @click="tag=16" :class="{active:tag==16}">韩国</span>
    </div>
    <!-- 底部的table -->
    <table class="el-table playlit-table">
      <thead>
        <th></th>
        <th></th>
        <th>音乐标题</th>
        <th>歌手</th>
        <th>专辑</th>
        <th>时长</th>
      </thead>
      <tbody>
        <tr class="el-table__row" v-for="(item,index) in list" :key="index">
          <td>{{index+1}}</td>
          <td>
            <div class="img-wrap">
              <img :src="item.album.picUrl" alt="" />
              <span class="iconfont icon-play" @click="playMusic(item.id)"></span>
            </div>
          </td>
          <td>
            <div class="song-wrap">
              <div class="name-wrap">
                <span>{{item.name}}</span>
                <span class="iconfont icon-mv"></span>
              </div>
              <span>{{item.alias[0]}}</span>
            </div>
          </td>
          <td >{{item.artists[0].name}}</td>
          <td>{{item.album.name}}</td>
          <td>{{item.duration | format('mm:ss')}}</td>
        </tr>
      </tbody>
    </table>
    <!-- 分页器 -->
    <el-pagination
      @current-change="handleCurrentChange"
      background
      layout="prev, pager, next"
      :total="total"
      :current-page="page"
      :page-size="20"
    >
    </el-pagination>
  </div>
</template>

<script>

 
export default {
  name: 'songs',
  data() {
    return {
      //最新音乐
      list:[],
      tag:'0',
      total:0,
      page:0
    };
  },
  created() {
      this.getList()
  },
  methods: {
    getList(){
      this.$axios({
      url:'https://autumnfish.cn/top/song',
      method:'get',
      params:{
        type:this.tag,
        limit: 10,

      }
    }).then((res)=>{
      // console.log(res)
      this.list = res.data.data
    })
    },
    playMusic(id){
      this.$axios({
      url:'https://autumnfish.cn/song/url',
      method:'get',
      params:{
        id
      }
    }).then((res)=>{
      // console.log(res.data.data[0])
      const {url} = res.data.data[0]
        this.$parent.musicUrl = url
    })
    },
    handleCurrentChange(val){
        console.log(val)
    }
  },
  watch: {
    tag(){
      this.getList()
    }
  },
};
</script>

<style >

</style>
