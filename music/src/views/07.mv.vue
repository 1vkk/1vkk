<template>
  <div class="mv-container">
    <div class="mv-wrap">
      <h3 class="title">mv详情</h3>
      <!-- mv -->
      <div class="video-wrap">
        <video
          controls
          :src="mvUrl"
        ></video>
      </div>
      <!-- mv信息 -->
      <div class="info-wrap">
        <div class="singer-info">
          <div class="avatar-wrap">
            <img :src="url" alt="" />
          </div>
          <span class="name">{{ infomv.artistName}}</span>
        </div>
        <div class="mv-info">
          <h2 class="title">{{ infomv.name }}</h2>
          <span class="date">发布：{{ infomv.publishTime}}</span>
          <span class="number">播放：{{ infomv.playCount }}</span>
          <p class="desc">
            {{ infomv.desc}}
          </p>
        </div>
      </div>
      <!-- 精彩评论 -->
      <div class="comment-wrap">
        <p class="title">精彩评论<span class="number">(666)</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item,index) in hotComments" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{ item.user.nickname }}：</span>
                <span class="comment">{{ item.content }}</span>
              </div>
              <div class="re-content" v-if="item.beReplied.length!=0">
                <span class="name">{{ item.beReplied[0].user.nickname }}</span>
                <span class="comment">{{ item.beReplied[0].content }}</span>
              </div>
              <div class="date">{{ item.timeStr }}</div>
            </div>
          </div>
        </div>
      </div>
      <!-- 最新评论 -->
      <div class="comment-wrap">
        <p class="title">最新评论<span class="number">(666)</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item,index) in comments" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{ item.user.nickname}}:</span>
                <span class="comment">{{ item.content }}</span>
              </div>
              <div class="re-content" v-if="item.beReplied.length!=0">
                <span class="name">{{ item.beReplied[0].user.nickname }}:</span>
                <span class="comment">{{ item.beReplied[0].content }}</span>
              </div>
              <div class="date">{{ item.timeStr }}</div>
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
          <div class="item" v-for="(item,index) in mvs" :key="index" @click="toPlayMv(item.id)">
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{ item.playCount}}</div>
              </div>
              <span class="time">{{ item.duration }}</span>
            </div>
            <div class="info-wrap">
              <div class="name">{{ item.name }}</div>
              <div class="singer">{{ item.artistName}}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'mv',
  data() {
    return {
      // mv在线地址
      mvUrl: '',
      // 相关mv
      mvs: [],
      // mv信息
      infomv: {},
      url: '',
      // 总条数
      total: 20,
      // 页码
      page: 1,
      // 页容量
      limit: 10,
      // 热门评论
      hotComments:{},
      // 评论条数
      Count: '',
      // 最新评论
      comments: []
    };
  },
  created(){
    this.playMV()
    this.aboutMv()
    this.infoMv()
    this.info()
  },
  methods: {
    // 进入相关MV页面
    async toPlayMv(id){
      const { data:res } = await axios.get('/mv/url',{
        params:{
          id
        }
      })
      // console.log(res)
      this.mvUrl = res.data.url
      // console.log(id)
    },
    // mv评论
    async info(){
      const { data:res } =await axios.get('/comment/mv',{
        params:{
          id: this.$route.query.q,
          limit: this.limit,
          offset: (this.page-1)*this.limit
        }
      })
      console.log(res)
      this.hotComments = res.hotComments
      this.Count = res.total
      this.comments = res.comments
      // console.log(this.hotComments)
    },
    // 获取mv信息
    async infoMv(){
      const { data:res } = await axios.get('/mv/detail',{
        params:{
          mvid: this.$route.query.q
        }
      })
      // console.log(res)
      this.infomv = res.data
      // 获取歌手头像
      const { data:res1 } = await axios.get('/artists',{
        params:{
          id: res.data.artists[0].id
        }
      })
      // console.log(res1)
      this.url = res1.artist.picUrl
    },
    async playMV(){
      const { data:res } = await axios.get('/mv/url',{
        params:{
          id: this.$route.query.q
        }
      })
      // console.log(res)
      this.mvUrl = res.data.url
    },
    // 获取相关mv
    async aboutMv(){
      const { data:res } = await axios.get('/simi/mv',{
        params: {
          mvid: this.$route.query.q
        }
      })
      // console.log(res)
      this.mvs = res.mvs
      for(let i = 0; i<this.mvs.length; i++){
          if(this.mvs[i].playCount>10000){
            this.mvs[i].playCount = parseInt((this.mvs[i].playCount)/10000) + '万'
          }
          let min = parseInt(this.mvs[i].duration/1000/60)
          let sec = parseInt(this.mvs[i].duration/1000%60)
        if(min<10){
          min = '0' + min
        }
        if(sec<10){
          sec = '0' + sec
        }
        this.mvs[i].duration = `${min}:${sec}`
        }
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.page = val
      axios.get('/comment/mv',{
        params:{
          id: this.$route.query.q,
          limit: this.limit,
          offset: (this.page-1)*this.limit
        }
      }).then(res=>{
        console.log(res)
        this.hotComments = res.data.hotComments
        this.Count = res.data.total
        this.comments = res.data.comments
      })
      // console.log(this.hotComments)
    },
    }
  }
</script>

<style></style>
