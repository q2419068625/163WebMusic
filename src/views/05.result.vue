<template>
  <div class="result-container">
    <div class="title-wrap">
      <h2 class="title">{{ this.$route.query.keywords }}</h2>
      <span class="sub-title">找到{{ songCount }}个结果</span>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲" name="songs">
        <table class="el-table">
          <thead>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr
              class="el-table__row"
              v-for="(item, index) in list"
              :key="index"
              @dblclick="playMusic(item.id)"
            >
              <td>{{ index + 1 }}</td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{ item.name }}</span>
                    <span class="iconfont icon-mv" v-if="item.mvid != 0"></span>
                  </div>
                  <span v-if="item.alias.length != 0">{{ item.alias[0] }}</span>
                </div>
              </td>
              <td>{{ item.artists[0].name }}</td>
              <td>{{ item.album.name }}</td>
              <td>{{ item.duration | format("mm:ss") }}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="歌单" name="lists">
        <div class="items">
          <div class="item" v-for="(item, index) in playlists" :key="index">
            <div class="img-wrap">
              <div class="num-wrap">
                播放量:
                <span class="num">{{ item.playCount }}</span>
              </div>
              <img :src="item.coverImgUrl" alt="" />
              <span class="iconfont icon-play" @click="toPlayList(item.id)"></span>
            </div>
            <p class="name">{{ item.name }}</p>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="MV" name="mv">
        <div class="items mv">
          <div class="item" v-for="(item,index) in mvs" :key="index">
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play" @click="toMvs(item.id)"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{item.playCount}}</div>
              </div>
              <span class="time">{{item.duration|format('mm:ss')}}</span>
            </div>
            <div class="info-wrap">
              <div class="name">{{item.name}}</div>
              <div class="singer">{{item.artistName}}</div>
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
export default {
  name: "result",
  data() {
    return {
      activeIndex: "songs",
      //关键词
      keywords: "",
      //返回数量
      limit: 10,
      //偏移数量
      offset: 0,
      //搜索类型
      type: 1,
      //搜索结果
      list: [],
      //歌曲数量
      songCount: 0,
      //歌单数据
      playlists: [],
      //mvs
      mvs:[]
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      this.$axios({
        url: "https://autumnfish.cn/search",
        method: "get",
        params: {
          keywords: this.$route.query.keywords,
          limit: this.limit,
          offset: this.offset,
          type: this.type,
        },
      }).then((res) => {
        console.log(res);
        const { songs, songCount, playlists, playlistCount ,mvs,mvCount} = res.data.result;
        //歌曲结果
        //歌曲数量
        //歌单数据
        if (this.type == 1) {
          this.songCount = songCount;
           this.list = songs;
        } else if (this.type == 1000) {
          this.songCount = playlistCount;
          this.playlists = playlists;
          for (let i = 0; i < this.playlists.length; i++) {
            if (this.playlists[i].playCount > 100000) {
              this.playlists[i].playCount =
                parseInt(this.playlists[i].playCount / 10000) + "万";
            }
          }
        }else if(this.type ==1004){
          this.mvs = mvs
          this.songCount = mvCount;
          for(let j=0;j<this.mvs.length;j++){
            if(this.mvs[j].playCount > 100000){
              this.mvs[j].playCount =
                parseInt(this.mvs[j].playCount / 10000) + "万";
            }
          }
        }
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
    },
    toPlayList(id){
      this.$router.push('/playlist?id='+id)
    },
    toMvs(id){
        this.$router.push('/mv?id='+id)
    }
  },
  watch: {
    activeIndex() {
      switch (this.activeIndex) {
        //歌曲
        case "songs":
          this.type = 1;
          break;
        //歌单
        case "lists":
          this.type = 1000;
          break;
        //mv
        case "mv":
          this.type = 1004;
          this.limit = 8
          break;
        default:
          break;
      }
      this.getList();
    },
  },
};
</script>

<style></style>
