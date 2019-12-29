<template>
  <div id="app">
    <p>나의 아이디는 {{socket.id}}입니다.</p>
    <!-- <p>나의 역활은 {{myRole}}입니다.</p> -->
    <p>현재 {{currentOrder}}의 턴입니다.</p>
    <div>
      <input placeholder="아이피입력" @keydown.enter="join" v-model="ip">
      <button @click="join">입력</button>
      <button @click="leave">연결끊기</button>
      <button v-if="isKing" @click="gameStart">시작</button>
      멤버 : 총 {{userList.length}}명
      
      <br>
      <!-- 덱 : {{deck}} -->
    </div>
    <div>
      <h3>총 {{turn+1}}턴 째</h3>
      <h3> {{round + 1}}라운드 {{(turn - round*users.length)+1}}턴 째</h3>
    </div>
    <div v-if="pickedCards.length>0">
      <h2>뽑힌카드</h2>
      <h3>{{pickedCards}}</h3>
    </div>
    <div
      v-for="(user, i) in users"
      :key="i"
      :style="user.id==socket.id?'color:red':''"
      style="margin:5px; border:solid"
    >
      <h2>{{user.id}}</h2>
      <h2>{{i+1}}번 플레이어 - 역할 : {{user.id==socket.id?user.role:'???'}}</h2>
      <p 
        v-for="(card, j) in user.deck"
        :key="j"
        @click="pick(i,j)"
      >
        [[ {{user.id==socket.id?card:'???'}} ]]
      </p>
    </div>
    <!-- <others-deck 
      v-for="(d,i) in othersDeck"
      :key="i"
      :deck="d"
      @select-card="selectOthersCard"
    /> -->
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
      myRole: '',
      socket: '',
      ip: '127.0.0.1',
      isKing: false,
      userList: [],

      round :0,
      turn :0,
      currentOrder :'',
      users: [],
      pickedCards: [],
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
          
          if(this.userList[0]==this.socket.id){
            this.isKing=true
          }
          else{
            this.isKing=false
          }
        });

        this.socket.on('game', (data) => {
          this.round = data.round
          this.turn = data.turn
          this.currentOrder = this.userList[data.currentOrder]
          this.pickedCards = data.pickedCards
          this.users = data.users
        })

        //종료
        this.socket.on('gameEnd', (data) => {
          alert(data)
          //여기서 게임판 초기화 할까???
        })
      }
    },
    gameStart(){
      this.socket.emit('gameStart','start')
    },
    pick(i,j){
      if(this.currentOrder != this.socket.id){
        alert('내 차례가 아닙니다.')
      }
      else if(this.userList[i]==this.socket.id){
        alert('다른 플레이어의 카드를 선택하세요')
      }
      else{
        this.socket.emit('pick',{i,j})
      }
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
