<template>
  <div class="playlist-container">
    <div class="top-wrap">
      <div class="img-wrap">
        <img :src="list.coverImgUrl" alt="" />
      </div>
      <div class="info-wrap">
        <p class="title">{{ list.name }}</p>
        <div class="author-wrap">
          <img
            class="avatar"
            v-if="list.creator"
            :src="list.creator.avatarUrl"
            alt=""
          />
          <span class="name" v-if="list.creator">{{
            list.creator.nickname
          }}</span>
          <span class="time"
            >{{ list.createTime | format("YYYY-MM-DD") }} 创建</span
          >
        </div>
        <div class="play-wrap">
          <span class="iconfont icon-circle-play" @click="playMusic(list.id)"></span>
          <span class="text">播放全部</span>
        </div>
        <div class="tag-wrap">
          <span class="title">标签:</span>
          <ul>
            <li v-for="(item, index) in list.tags" :key="index">{{ item }}</li>
          </ul>
        </div>
        <div class="desc-wrap">
          <span class="title">简介:</span>
          <span class="desc">{{ list.description }}</span>
        </div>
      </div>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲列表" name="1">
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
            <tr
              class="el-table__row"
              v-for="(item, index) in list.tracks"
              :key="index"
            >
              <td>{{ index + 1 }}</td>
              <td>
                <div class="img-wrap">
                  <img :src="item.al.picUrl" alt="" />
                  <span class="iconfont icon-play" @click="playMusic(item.id)"></span>
                </div>
              </td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{ item.name }}</span>
                    <span class="iconfont icon-mv"></span>
                  </div>
                  <span v-if="item.al.tns.length != 0">{{
                    item.al.tns[0]
                  }}</span>
                </div>
              </td>
              <td>{{ item.ar[0].name }}</td>
              <td>{{ item.al.name }}</td>
              <td>{{ item.dt | format("mm:ss") }}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane :label="`评论(${total})`" name="2">
        <!-- 精彩评论 -->
        <div class="comment-wrap">
          <p class="title">精彩评论<span class="number">({{hotCount}})</span></p>
          <div class="comments-wrap">
            <div class="item" v-for="(item,index) in hotComments" :key="index">
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{item.user.nickname}}：</span>
                  <span class="comment">{{item.content}}</span>
                </div>
                <div class="re-content" v-if="item.beReplied.length!=0">
                  <span class="name"  >{{item.beReplied[0].user.nickname}}：</span>
                  <span class="comment"  >{{item.beReplied[0].content}}</span>
                </div>
                <div class="date">{{item.time|format('YYYY-MM-DD hh:mm:ss')}}</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 最新评论 -->
        <div class="comment-wrap">
          <p class="title">最新评论<span class="number">({{total}})</span></p>
          <div class="comments-wrap">
            <div class="item" v-for="(item,index) in comments" :key="index">
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{item.user.nickname}}：</span>
                  <span class="comment">{{item.content}}</span>
                </div>
                <div class="re-content" v-if="item.beReplied.length!=0">
                  <span class="name">{{item.beReplied[0].user.nickname}}：</span>
                  <span class="comment">{{item.beReplied[0].content}}</span>
                </div>
                <div class="date">{{item.time|format('YYYY-MM-DD hh:mm:ss')}}</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 分页器 -->
        <el-pagination
          @current-change="handleCurrentChange"
          background
          layout="prev, pager, next"
          :total="total"
          :current-page="page"
          :page-size="5"
        >
        </el-pagination>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
export default {
  name: "playlist",
  data() {
    return {
      activeIndex: "1",
      // 总条数
      total: 0,
      // 页码
      page: 1,
      list: {},
      //热门评论
      hotComments:[],
      //热门评论个数
      hotCount:0,
      //最新评论
      comments:[]
    };
  },
  methods: {
    handleCurrentChange(val) {
      // console.log(`当前页: ${val}`);
      this.page = val
      this.getComments()
    },
     getComments(){
      this.$axios({
      url: "https://autumnfish.cn/comment/playlist",
      method: "get",
      params: {
        id: this.$route.query.id,
        limit:10,
        offset:(this.page-1)*10
      },
    }).then((res) => {
      // console.log(res);
      this.total = res.data.total
      this.comments = res.data.comments
    });
    },
    playMusic(id) {
      this.$axios({
        url: "https://autumnfish.cn/song/url",
        method: "get",
        params: {
          id,
        },
      }).then((res) => {
        const url = res.data.data[0].url;
        //子组件传值给父组件
        this.$parent.musicUrl = url;
        // console.log(res)
      });
    }
  },
  created() {
    //歌单信息
    this.$axios({
      url: "https://autumnfish.cn/playlist/detail",
      method: "get",
      params: {
        id: this.$route.query.id,
      },
    }).then((res) => {
      // console.log(res);
      this.list = res.data.playlist;
    });
    //热门评论
    this.$axios({
      url: "https://autumnfish.cn/comment/hot",
      method: "get",
      params: {
        id: this.$route.query.id,
        type:2,
        limit:10
      },
    }).then((res) => {
      // console.log(res);
      this.hotComments = res.data.hotComments
      this.hotCount = res.data.total
    });
    //最新评论
    this.getComments()
  },
  
};
</script>

<style></style>
