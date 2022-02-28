<template>
  <div class="songs-container">
    <div class="tab-bar">
      <span class="item" :class="{active:tag=='0'}" @click="tag='0'">全部</span>
      <span class="item" :class="{active:tag=='7'}" @click="tag='7'">华语</span>
      <span class="item" :class="{active:tag=='96'}" @click="tag='96'">欧美</span>
      <span class="item" :class="{active:tag=='8'}" @click="tag='8'">日本</span>
      <span class="item" :class="{active:tag=='16'}" @click="tag='16'">韩国</span>
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
          <td>{{ index+1 }}</td>
          <td>
            <div class="img-wrap">
              <img :src="item.picUrl" alt="" />
              <span class="iconfont icon-play" @click="playMusic(item.id)"></span>
            </div>
          </td>
          <td>
            <div class="song-wrap">
              <div class="name-wrap">
                <span>{{ item.name }}</span>
                <span class="iconfont icon-mv"></span>
              </div>
              <span></span>
            </div>
          </td>
          <td>{{ item.song.album.artists[0].name}}</td>
          <td>{{ item.song.album.name }}</td>
          <td>{{item.song.duration}}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'songs',
  data() {
    return {
      list: [],
      tag: '0'
    }
  },
  watch:{
    // 监听tag变化
    tag(){
      this.getList()
    }
  },
  created(){
    this.getList()
  },
  methods:{
    // 获取歌曲列表
    getList(){
      axios({
      method: 'get',
      url: '/personalized/newsong',
      params:{
        type: this.tag
      }
    }).then(res=>{
      // console.log(res)
      console.log(this.tag)
      // 赋值给数组
      this.list = res.data.result
      for( let i=0; i < this.list.length; i++){
        let duration = this.list[i].song.duration
        let min = parseInt(duration / 1000 / 60)
        if(min < 10){
          min = '0' + min
        }
        let sec = parseInt((duration / 1000) % 60)
        if(sec < 10){
          sec = '0' + sec
        }
        this.list[i].song.duration = `${min}:${sec}`
        // console.log(min + '|' + sec)
      }
    })
    },
    async playMusic(id){
      const { data:res } = await axios.get('/song/url',{
        params: {
          id
        }
      })
      // console.log(res)
      let url = res.data[0].url
      this.$parent.musicUrl = url
    }
  }
};
</script>

<style >

</style>
