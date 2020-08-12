<template>
  <div class="mv-container">
    <div class="mv-wrap">
      <h3 class="title">mv详情</h3>
      <!-- mv -->
      <div class="video-wrap">
        <video controls :src="url"></video>
      </div>
      <!-- mv信息 -->
      <div class="info-wrap">
        <div class="singer-info">
          <div class="avatar-wrap">
            <img :src="mvInfo.cover" alt="" />
          </div>
          <span
            class="name"
            v-for="(item, index) in mvInfo.artists"
            :key="index"
          >
            {{ item.name
            }}<span v-if="mvInfo.artists.length - 1 != index">/</span>
          </span>
        </div>
        <div class="mv-info">
          <h2 class="title">{{ mvInfo.name }}</h2>
          <span class="date">发布：{{ mvInfo.publishTime }}</span>
          <span class="number">播放：{{ mvInfo.playCount }}次</span>
          <p class="desc">
            {{ mvInfo.desc }}
          </p>
        </div>
      </div>
      <!-- 精彩评论 -->
      <div class="comment-wrap" v-if="hotComments">
        <p class="title">
          精彩评论<span class="number">({{ hotComments.length }})</span>
        </p>
        <div class="comments-wrap">
          <div class="item" v-for="(item, index) in hotComments" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{ item.user.nickname }}：</span>
                <span class="comment">{{ item.content }}</span>
              </div>
              <div class="re-content" v-if="item.beReplied.length != 0">
                <span class="name"
                  >{{ item.beReplied[0].user.nickname }}：</span
                >
                <span class="comment">{{ item.beReplied[0].content }}</span>
              </div>
              <div class="date">
                {{ item.time | format("YYYY-MM-DD hh:mm:ss") }}
              </div>
            </div>
          </div>
        </div>
      </div>
      <!-- 最新评论 -->
      <div class="comment-wrap">
        <p class="title">
          最新评论<span class="number">({{ total }})</span>
        </p>
        <div class="comments-wrap">
          <div class="item" v-for="(item, index) in comments" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{ item.user.nickname }}：</span>
                <span class="comment">{{ item.content }}</span>
              </div>
              <div class="re-content" v-if="item.beReplied.length != 0">
                <span class="name"
                  >{{ item.beReplied[0].user.nickname }}：</span
                >
                <span class="comment">{{ item.beReplied[0].content }}</span>
              </div>
              <div class="date">
                {{ item.time | format("YYYY-MM-DD hh:mm:ss") }}
              </div>
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
        :limit="limit"
      >
      </el-pagination>
    </div>
    <div class="mv-recommend">
      <h3 class="title">相关推荐</h3>
      <div class="mvs">
        <div class="items">
          <div class="item" v-for="(item, index) in mvs" :key="index">
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play" @click="playMvs(item.id)"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{ item.playCount }}</div>
              </div>
              <span class="time">{{ item.duration | format("mm:ss") }}</span>
            </div>
            <div class="info-wrap">
              <div class="name">{{ item.name }}</div>
              <div class="singer">{{ item.artists[0].name }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "mv",
  data() {
    return {
      // 总条数
      total: 20,
      // 页码
      page: 1,
      // 页容量
      limit: 10,
      //MV地址
      url: "",
      mvInfo: {},
      //mv推荐
      mvs: [],
      //最新评论
      comments: [],
      //精彩评论
      hotComments: [],
    };
  },
  methods: {
    handleCurrentChange(val) {
      // console.log(`当前页: ${val}`);
      this.page = val;
      this.getComments();
    },
    //MV评论
    getComments() {
      this.$axios({
        url: " https://autumnfish.cn/comment/mv",
        method: "get",
        params: {
          id: this.$route.query.id,
          limit: this.limit,
          offset: (this.page - 1) * this.limit,
        },
      }).then((res) => {
        // console.log(res);
        const { hotComments, comments, total } = res.data;
        this.hotComments = hotComments;
        this.comments = comments;
        this.total = total;
      });
    },
    //MV地址
    getMVUrl() {
      this.$axios({
        url: "https://autumnfish.cn/mv/url",
        method: "get",
        params: {
          id: this.$route.query.id,
        },
      }).then((res) => {
        // console.log(res)
        this.url = res.data.data.url;
      });
    },
    //相关推荐
    getMvs() {
      this.$axios({
        url: "https://autumnfish.cn/simi/mv",
        method: "get",
        params: {
          mvid: this.$route.query.id,
        },
      }).then((res) => {
        // console.log(res);
        this.mvs = res.data.mvs;
        for (let i = 0; i < this.mvs.length; i++) {
          if (this.mvs[i].playCount > 100000) {
            this.mvs[i].playCount =
              parseInt(this.mvs[i].playCount / 10000) + "万";
          }
        }
      });
    },
    //MV信息
    getMvInfo() {
      this.$axios({
        url: " https://autumnfish.cn/mv/detail",
        method: "get",
        params: {
          mvid: this.$route.query.id,
        },
      }).then((res) => {
        // console.log(res);
        this.mvInfo = res.data.data;
        if (this.mvInfo.playCount > 100000) {
          this.mvInfo.playCount =
            parseInt(this.mvInfo.playCount / 10000) + "万";
        }
      });
    },
    playMvs(id) {
      this.$axios({
        url: " https://autumnfish.cn/comment/mv",
        method: "get",
        params: {
          id,
          limit: this.limit,
          offset: (this.page - 1) * this.limit,
        },
      }).then((res) => {
        // console.log(res);
        const { hotComments, comments, total } = res.data;
        this.hotComments = hotComments;
        this.comments = comments;
        this.total = total;
      });
      this.$axios({
        url: "https://autumnfish.cn/mv/url",
        method: "get",
        params: {
          id,
        },
      }).then((res) => {
        // console.log(res)
        this.url = res.data.data.url;
      });
      this.$axios({
        url: "https://autumnfish.cn/simi/mv",
        method: "get",
        params: {
          mvid: id,
        },
      }).then((res) => {
        // console.log(res);
        this.mvs = res.data.mvs;
        for (let i = 0; i < this.mvs.length; i++) {
          if (this.mvs[i].playCount > 100000) {
            this.mvs[i].playCount =
              parseInt(this.mvs[i].playCount / 10000) + "万";
          }
        }
      });

      this.$axios({
        url: " https://autumnfish.cn/mv/detail",
        method: "get",
        params: {
          mvid: id,
        },
      }).then((res) => {
        console.log(res);
        this.mvInfo = res.data.data;
        if (this.mvInfo.playCount > 100000) {
          this.mvInfo.playCount =
            parseInt(this.mvInfo.playCount / 10000) + "万";
        }
      });
    },
    
  },
  created() {
    //MV评论
    this.getComments()
    //MV地址
    this.getMVUrl()
    //相关推荐
    this.getMvs()
    //MV信息
    this.getMvInfo()
  },
};
</script>

<style></style>
