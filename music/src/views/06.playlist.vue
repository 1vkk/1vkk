<template>
  <div class="playlist-container">
    <div class="top-wrap">
      <div class="img-wrap">
        <img :src="playLists.coverImgUrl" alt="" />
      </div>
      <div class="info-wrap">
        <p class="title">{{ playLists.name}}</p>
        <div class="author-wrap">
          <img class="avatar" :src="playLists.creator.avatarUrl" alt="" />
          <span class="name">{{ playLists.creator.nickname }}</span>
          <span class="time">{{ playLists.createTime }} 创建</span>
        </div>
        <div class="play-wrap">
          <span class="iconfont icon-circle-play"></span>
          <span class="text">播放全部</span>
        </div>
        <div class="tag-wrap">
          <span class="title">标签:</span>
          <ul>
            <li v-for="(item,index) in playLists.tags" :key="index">{{ item }}</li>
          </ul>
        </div>
        <div class="desc-wrap">
          <span class="title">简介:</span>
          <span class="desc"
            >{{ playLists.description }}</span
          >
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
            <tr class="el-table__row" v-for="(item,index) in playLists.tracks" :key="index" @click="playMusic(item.id)">
              <td>{{ index+1 }}</td>
              <td>
                <div class="img-wrap">
                  <img :src="item.al.picUrl" alt="" />
                  <span class="iconfont icon-play"></span>
                </div>
              </td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{ item.al.name}}</span>
                    <span class="iconfont icon-mv"></span>
                  </div>
                  <span>电视剧加油练习生插曲</span>
                </div>
              </td>
              <td>{{ item.ar[0].name}}</td>
              <td>{{ item.name }}</td>
              <td>06:03</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane v-if="total" :label="`评论${total}`" name="2">
        <!-- 精彩评论 -->
        <div class="comment-wrap">
          <p class="title">精彩评论<span class="number">({{ hotCount }})</span></p>
          <div class="comments-wrap">
            <div class="item" v-for="(item,index) in hotComment" :key="index">
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{ item.user.nickname }}</span>
                  <span class="comment">{{ item.content }}</span>
                </div>
                <div class="re-content" v-if="item.beReplied.length!=0">
                  <span class="name">{{ item.beReplied[0].user.nickname }}</span>
                  <span class="comment">{{ item.beReplied[0].content }}</span>
                </div>
                <div class="date">{{ item.timeStr }} 17:26:11</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 最新评论 -->
        <div class="comment-wrap">
          <p class="title">最新评论<span class="number">({{total}})</span></p>
          <div class="comments-wrap">
            <div class="item" v-for="(item,index) in comment" :key="index">
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{ item.user.nickname }}</span>
                  <span class="comment">{{ item.content }}</span>
                </div>
                <div class="re-content" v-if="item.beReplied.length!=0">
                  <span class="name">{{ item.beReplied[0].user.nickname }}</span>
                  <span class="comment">{{ item.beReplied[0].content }}</span>
                </div>
                <div class="date">2020-02-12 17:26:11</div>
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
import axios from 'axios';
export default {
  name: 'playlist',
  data() {
    return {
      activeIndex: '1',
      // 总条数
      total: 0,
      // 页码
      page: 1,
      playLists: {}, //歌单信息
      hotComment: [], //热门评论
      hotCount: '', //热门评论的个数
      comment: [], // 最新评论
      url: '', //音乐链接
    };
  },
  created(){
    this.playList()
      // 获取热门评论
    axios({
      url: '/comment/hot',
      method: 'get',
      params: {
        id: this.$route.query.q,
        type: 2
      }
    }).then(res=>{
      // console.log(res)
      this.hotComment = res.data.hotComments
      this.hotCount = res.data.total
    })
    // 获取最新评论
    axios.get('/comment/playlist',{
      params: {
        id: this.$route.query.q,
        limit: 20,
        offset: 0
      }
    }).then(res=>{
      // console.log(res)
      this.comment = res.data.comments
      this.total = res.data.total
    })
  },
  methods: {
    // 点击播放音乐
    playMusic(id){
      axios({
        url: '/song/url',
        method: 'get',
        params: {
          id
        }
      }).then(res=>{
        console.log(res)
        this.url = res.data.data[0].url
        this.$parent.musicUrl = this.url
      })
    },
    playList(){
      axios({
        method: 'get',
        url: '/playlist/detail',
        params:{
          id: this.$route.query.q
        }
      }).then(res=>{
      // console.log(res)
      this.playLists = res.data.playlist
      // console.log(this.playLists)
      })
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.page = val
      axios.get('/comment/playlist',{
      params: {
        id: this.$route.query.q,
        limit: 20,
        offset: (this.page - 1) * this.limit
      }
    }).then(res=>{
      // console.log(res)
      this.comment = res.data.comments
      this.total = res.data.total
    })
    }
  }
};
</script>

<style >

</style>
