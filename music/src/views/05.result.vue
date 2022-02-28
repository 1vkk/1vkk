<template>
  <div class="result-container">
    <div class="title-wrap">
      <h2 class="title">{{ $route.query.q }}</h2>
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
            <tr class="el-table__row" v-for="(item,index) in list" :key="index" @click="playMusic(item.id)">
              <td>{{index+1}}</td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{ item.name }}</span>
                    <span class="iconfont icon-mv"></span>
                  </div>
                  <span>{{ item.alias[0] }}</span>
                </div>
              </td>
              <td>{{ item.artists[0].name}}</td>
              <td>{{ item.album.name}}</td>
              <td>{{ item.duration }}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="歌单" name="lists" >
        <div class="items">
          <div class="item" v-for="(item,index) in playLists" :key="index" @click="toPlaylist(item.id)">
            <div class="img-wrap">
              <div class="num-wrap">
                播放量:
                <span class="num">{{ item.playCount}}</span>
              </div>
              <img :src="item.coverImgUrl" alt="" />
              <span class="iconfont icon-play"></span>
            </div>
            <p class="name">{{ item.name}}</p>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="MV" name="mv">
        <div class="items mv">
          <div class="item" v-for="(item,index) in mvs" :key="index" @click="toMV(item.id)">
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
              <div class="name">{{ item.name}}</div>
              <div class="singer">{{ item.artistName}}</div>
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'result',
  data() {
    return {
      activeIndex: 'songs',
      // 歌曲数据
      list: [],
      // 歌单数据
      playLists: [],
      // MV数据
      mvs: [],
      // 搜索结果总数
      songCount: 0,
      type: 1000
    };
  },
  watch:{
    async activeIndex(){
      let type = 1
      let limit = 10
      switch(this.activeIndex){
        case 'songs':
          type = 1
          limit = 10
          break;
        case 'lists':
          type = 1000
          limit = 10
          break;
        case 'mv':
          type = 1004
          limit = 8
          break;
        default:
          break
      }
      const { data:res } = await axios.get('https://autumnfish.cn/search',{
        params:{
          keywords: this.$route.query.q,
          type,
          limit
        }
      })
      if(type==1){
        this.list = res.result.songs
        this.songCount = res.result.songCount
       for(let i = 0; i<this.list.length; i++){
        let min = parseInt(this.list[i].duration/1000/60)
        let sec = parseInt(this.list[i].duration/1000%60)
        if(min<10){
          min = '0' + min
        }
        if(sec<10){
          sec = '0' + sec
        }
        this.list[i].duration = `${min}:${sec}`
        // console.log(min + '|' + sec)
      }
      }else if(type==1000){
        // console.log(res)
        this.playLists = res.result.playlists
        this.songCount = res.result.playlistCount
        for(let i = 0; i<this.playLists.length; i++){
          if(this.playLists[i].playCount>10000){
            this.playLists[i].playCount = parseInt((this.playLists[i].playCount)/10000) + '万'
          }
        }
      }else{
        // console.log(res)
        this.mvs = res.result.mvs
        this.songCount = res.result.mvCount
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
      }
    }
  },
  created(){
    this.playList()
  },
  methods:{
    toMV(id){
      this.$router.push(`mv?q=${id}`)
    },
    toPlaylist(id){
      this.$router.push(`playlist?q=${id}`)
    },
    async playList(){
      const { data:res } = await axios.get('/search',{
        params:{
          keywords: this.$route.query.q,
          type: 1,
          limit: 10
        }
      })
      // console.log(res)
      this.list = res.result.songs
      this.songCount = res.result.songCount
      for(let i = 0; i<this.list.length; i++){
        let min = parseInt(this.list[i].duration/1000/60)
        let sec = parseInt(this.list[i].duration/1000%60)
        if(min<10){
          min = '0' + min
        }
        if(sec<10){
          sec = '0' + sec
        }
        this.list[i].duration = `${min}:${sec}`
        // console.log(min + '|' + sec)
      }
    },
    // 点击播放音乐
    async playMusic(id){
      axios({
        method: 'get',
        url: '/song/url',
        params: {
          id
        }
      }).then(res => {
        console.log(res)
        let url = res.data.data[0].url
        this.$parent.musicUrl = url
        console.log(this.$parent.musicUrl)
      })
    }
  }
};
</script>

<style >

</style>
