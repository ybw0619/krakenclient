<template>
  <div id="app">
    <p>나의 아이디는 {{socket.id}}입니다.</p>
    <p>현재 {{nowTurn}}의 턴입니다.</p>
    <div>
      <input placeholder="아이피입력" @keydown.enter="join" v-model="ip">
      <button @click="join">입력</button>
      <button @click="leave">연결끊기</button>
      <button v-if="isKing" @click="gameStart">시작</button>
      멤버 : 총 {{userList.length}}명
      
      <br>
      덱 : {{deck}}
    </div>
    <others-deck 
      v-for="(d,i) in othersDeck"
      :key="i"
      :deck="d"
      @select-card="selectOthersCard"
    />
    <textarea 
      v-model="messages"
      ref="chatWindow"
      style="width:600px; height:300px; resize: none;" 
      disabled
    />
    <br>
    <input placeholder="채팅입력" @keydown.enter="sendMessage" v-model="message">
  </div>
</template>
<script>
import io from 'socket.io-client';
import OthersDeck from '@/components/OthersDeck'

export default {
  components: {
    OthersDeck
  },
  mounted(){
    
  },
  data(){
    return{
      nowTurn: '',
      socket: '',
      ip: '',
      isKing: false,
      userList: [],
      deck: [],
      othersDeck: [],
      //내가 보낼 메세지
      message:'',
      //주고받은 전체 메세지
      messages:''
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
          console.log(data)
        })
        //호준 메세지 받기
        this.socket.on('chat', (data) => {
            this.messages += `${data}\n` //그냥 줄바꿈으로 구현
        })

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

        this.socket.on('others',(data)=>{
          console.log('others', data)
          if (data.user !== this.socket.id) {
            this.othersDeck.push(data)
          }
        })

        this.socket.on('turn-start', (id) => {
          this.nowTurn = id
        })

        this.socket.on('turn-end', (turn) => {
          // turn.id 선택한 사람의 아이디
          // turn.card 선택한 사람이 가지고있는 카드덱의 인덱스

        })

      }
    },
    gameStart(){
      this.socket.emit('gameStart','start')
    },
    selectOthersCard(i) {
      console.log(i)
      this.socket.emit('turn-select', {
        id: this.socket.id,
        selectCard: i
      })
    },
    leave(){
      this.socket.close()
      this.socket=null
      this.isKing=false
      this.userList=[]
    },
    //호준 채팅입력
    sendMessage(){
      //메세지가 있을때만 (빈메세지 도배 방지)
      if(this.message){
        this.socket.emit('chat', `${this.socket.id} : ${this.message}`)
        this.message=''
      }
    },
    autoScroll(){
      let chatWindow = chatWindow =this.$refs.chatWindow
      chatWindow.scrollTop = chatWindow.scrollHeight
    }
  },
  beforeDestroy(){
    this.leave()
  },
  watch:{
    //채팅을 받거나 보내거나 해서 전체 메세지가 변하면 오토스크롤 작동
    messages(){
      this.autoScroll()
    }
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
