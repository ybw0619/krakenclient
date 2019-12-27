<template>
  <div id="app">
    <div>
      <input placeholder="아이피입력" @keydown.enter="join" v-model="ip">
      <button @click="join">입력</button>
      <button @click="leave">연결끊기</button>
      <button v-if="isKing" @click="gameStart">시작</button>
      멤버 : 총 {{userList.length}}명
      
      <br>
      덱 : {{deck}}
    </div>
    
  </div>
</template>
<script>
import io from 'socket.io-client';
export default {
  mounted(){
    
  },
  data(){
    return{
      socket:'',
      ip:'',
      isKing:false,
      userList:[],
      deck:[]
    }
  },
  methods:{
    join(){
      //소켓이 비어있을때만..
      if(!this.socket){
        this.socket = io(`${this.ip}:3000`);
        // this.id = this.socket.id
        console.log(this.socket);
        
        this.socket.on('broadcast', (data) => {
            console.log(data);
        });

        this.socket.on('userList', (data) => {
          this.userList = data
          console.log(this.userList);
          
          if(this.userList[0].id==this.socket.id){
            this.isKing=true
          }
          else{
            this.isKing=false
          }
        });

        this.socket.on('deck',(data)=>{
          console.log('덱', data);
          this.deck = data
          
        })

      }
    },
    gameStart(){
      this.socket.emit('gameStart','start')
    },
    leave(){
      this.socket.close()
      this.socket=null
      this.isKing=false
      this.userList=[]
    }
  },
  beforeDestroy(){
    this.leave()
  }
}
</script>
<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
