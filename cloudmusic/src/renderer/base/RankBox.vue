<template>
  <div class="rankBox spCenter">
    <div class="rankContainer" v-for="item in rankList1" :key="item.id">
      <div v-loading="loading1" style="position:absolute;width:100%;height:100%;background:white;" v-if="loading1"></div>
      <img v-lazy="item.coverImgUrl" class="rkPic"/>
        <ul class="rkSongList">
          <li class="rkSong alignCenter" v-for="(item1,index) in item.songList" :key="item1.mid" @dblclick="playSong(item1)">
            <p class="rKindex">{{index+1}}</p>
            <p class="rkName">{{item1.name}}</p>
            <p class="rkSinger" @click="toSinger(item1)">{{item1.singer}}</p>
          </li>
        </ul>
        <p class="rkAll" @click="getData(item)">
          查看全部>
        </p>
    </div>
    <div class="rankContainer" v-for="item in specailList" :key="item.id">
      <div v-loading="loading2" style="position:absolute;width:100%;height:100%;background:white;" v-if="loading2"></div>
      <img v-lazy="item.coverUrl" class="rkPic"/>
        <ul class="rkSongList" v-if="item.songList">
          <li class="rkSong alignCenter" v-for="(item1,index) in item.songList" :key="item1.id">
            <p class="rKindex">{{index+1}}</p>
            <p class="rkSinger" style="text-align:left;" @click="toSinger(item1)">{{item1.name}}</p>
          </li>
        </ul>
        <p class="rkAll" @click="$router.push('/singerrank')">
          查看全部>
        </p>
    </div> 
  </div>
</template>

<script>
import axios from 'axios'
import {Axios,getSongUrl,url} from '../common/api'
import {createSong} from '../common/song'
import {mapActions} from 'vuex'
export default {
  props:['list','specailList'],
  data() {
    return {
      loading1:true,
      loading2:true,
      rankList1:[],
      rankList2:[]
    }
  },
  mounted() {
    console.log(`${url}/top/list`)
    this.initSongList()

  },
  methods: {
    initSongList() {
      this.rankList1 = this.list.slice(0)
      this.rankList2 = this.specailList.slice(0)

      var promiseAll = this.rankList1.map((item)=>{
        return axios.get(`${url}/top/list`,{
          params:{
            idx:item.idx,
            timestamp: (new Date()).getTime()
          }
        })
      })
      axios.all(promiseAll).then((all) => {
        for(let i = 0; i < this.rankList1.length; i ++) {
          let songs = this._normalizeSongList(all[i].data.playlist.tracks.slice(0,8))
          
          this.$set(this.rankList1[i],'songList',songs)
          this.loading1 = false;
        }
      })
      for(let i = 0; i < this.rankList2.length; i ++) {
        axios.get(`${url}/toplist/artist`,{
          params:{
            timestamp: (new Date()).getTime()
          }
        }).then((result) => {
          let res = result.data
          if(res.code === 200) {
            this.$set(this.rankList2[i],'songList',res.list.artists.slice(0,8))
            this.loading2 = false
          }
        })
      }
    },
    _normalizeSongList(list) {
        let ret = []
        for(let i = 0; i < list.length; i ++) {    
            ret.push(createSong(list[i]))
        }
        return ret
    },
    playSong(song) {
        let url = ''
        let params = {
          id: song.mid
        }
        Axios(getSongUrl,params).then((res) => {
          url = res.data[0].url
          song.url = url
          this.insertSong(song)
        })
    },
    getData(data) {
      this.$emit("getData",data)
    },
    toSinger(song) {
      let id = song.id
      this.$router.push(`/singerDetail/${id}`)
    },
    ...mapActions([
      'insertSong'
    ])
  }
}
</script>

<style lang='scss'>
@import '../assets/css/base.scss';
.rankBox {
    width: 100%;
    height: 800px;
    flex-wrap: wrap;
    &::after {
          content: "";
          width: 30%;
        }
      .rankContainer {
        position:relative;
        border: 1px solid $borderColor;
        margin-bottom: 20px;
        width: 30%;
        .rkPic {
        width: 100%;
        height: 90px;
        background: red;
      }
      .rkSongList {
        width: 100%;
        height: 250px;
        .rkSong {
          cursor: default;
          font-size: 13px;
          padding: 5px 10px 8px 20px;
          &:nth-child(1) .rKindex,&:nth-child(2) .rKindex,&:nth-child(3) .rKindex{
            color:#CD2929;
          }
          .rKindex {
            font-size: 18px;
            margin-right: 20px;
            color:#666666;
          }
          .rkName {
            width: 60%;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
            color: #333333;
          }
          .rkSinger {
            width: 25%;
            color:#888888;
            text-align: right;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
            cursor: pointer;
            &:hover {
              color: #666666;
            }
          }
          &:hover {
            background: #EBECED;
          }
        }
        
      }
      .rkAll {
        cursor: pointer;
        float: right;
        font-size: 13px;
        color:#666666;
        margin-right: 10px;
        margin-top: 10px;
      }
    }
    
}
</style>