<template>
  <div class="lrcContainer">
    <div class="lrc" ref="lrc">
        <!-- {{ currentTimes }} - {{ durationTime }} -->
        <!-- 当前时间与下一次的时间差是当前文本要显示的时间长度 -->
        <!-- 当前时间：currentTimes  currentTimes+1 -->
        <!-- 当前时间：currentTimes >= allkeys[index] && currentTimes < allKeys[index+1] -->
        <!-- <p 
            v-if="currentTimes >= allkeys[index]  && currentTimes < allkeys[index+1]" 
            v-for="(value,key,index) in lrcData" 
            :key="index">
                {{ value }}
        </p> -->
        <p 
            :class="{'active':currentTimes >= allkeys[index]  && currentTimes < allkeys[index+1]}" 
            v-for="(value,key,index) in lrcData" 
            class="lrc-p" 
            :key="index">
                {{ value }}{{ getScrollTop(index) }}
        </p>
    </div>
  </div>
</template>
<script>
export default {
  name: "LRC",
  data() {
    return {
      lrcData: {},
      allkeys:[]
    };
  },
  props: {
    id: {
      type: [String, Number],
      default: 0
    },
    currentTimes: {
      type: [String, Number],
      default: 0
    },
    durationTime: {
      type: [String, Number],
      default: 0
    }
  },
  computed:{
      
  },
  mounted() {
    this.$axios
      .get(this.HOST + "/v1/restserver/ting", {
        params: {
          method: "baidu.ting.song.lry",
          songid: this.id
        }
      })
      .then(res => {
        // 数据解析
        // 25:"哪会怕有一天只你共我"
        this.setLRC(res.data);
      })
      .catch(error => {
        console.log(error);
      });
  },
  methods: {
    getAllKeys(){
        for(var i in this.lrcData){
            this.allkeys.push(i)
        }
    },
    setLRC(data) {
      var lyrics = data.lrcContent.split("\n");
      // 创建正则规则
      var lrcObj = {}; // 存储对象
      for (let i = 0; i < lyrics.length; i++) {
        var lyric = decodeURIComponent(lyrics[i]);
        var timeReg = /\[\d*:\d*((\.|\:)\d*)*\]/g; // [00:11.22]  [00:11:22]
        // 时间
        var timeRegExpArr = lyric.match(timeReg); // test
        if (!timeRegExpArr) continue; // break
        // 歌词
        var clause = lyric.replace(timeReg, "");
        // 时间字符串转化可对比的时间
        var t = timeRegExpArr[0];
        var min = Number(String(t.match(/\[\d*/i)).slice(1));
        var sec = Number(String(t.match(/\:\d*/i)).slice(1));
        var time = min * 60 + sec;
        lrcObj[time] = clause;
      }
      this.lrcData = lrcObj;
      this.getAllKeys();
    },
    // 滚动函数
    getScrollTop(index){
        if(this.currentTimes >= this.allkeys[index]  && this.currentTimes < this.allkeys[index+1]){
            this.$refs.lrc.style.top = -((index-2) * 30) +"px";
        }        
    }
  }
};
</script>
<style scoped>
.active {
  color: red;
}

.lrcContainer {
  width: 100%;
  height: 150px;
  overflow: hidden;
  position: relative;
}

.lrc {
  width: 100%;
  position: absolute;
  top: 0;
}

.lrc-p {
  height: 30px;
  line-height: 30px;
}

.up30 {
  margin-top: -30px;
}
</style>
