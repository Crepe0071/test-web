<template>
  <div>
    <p @click="method2">method2</p>
    <p @click="method3">method3</p>
    <img v-if="imageUrl" :src="imageUrl" alt="Loaded Image" />
  </div>
  <div>
    <h3>STOMP WebSocket 테스트 (Options API)</h3>
    <p>Connected: {{ isConnected }}</p>

    <div>
      <input v-model="sender" placeholder="Sender" />
      <input v-model="content" placeholder="Message" />
      <button @click="sendChat">Send</button>
    </div>

    <div v-if="messages.length">
      <h4>Received Messages:</h4>
      <ul>
        <li v-for="(msg, idx) in messages" :key="idx">
          {{ msg.sender }} : {{ msg.message }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import SockJS from 'sockjs-client'
import Stomp from 'stompjs'
import axios from "axios";

// Vue 3 Options API
export default {
  name: 'MyChat',
  data() {
    return {
      imageUrl: null, // 이미지를 저장할 변수
      // STOMP 클라이언트
      stompClient: null,
      // 연결 여부
      isConnected: false,
      // 수신된 메시지
      messages: [],
      // 사용자 입력
      sender: '',
      content: '',
    }
  },
  mounted() {
    // 컴포넌트가 마운트될 때 WebSocket(Stomp) 연결 시도
    this.connectStomp()
  },
  beforeUnmount() {
    // 언마운트 직전에 WebSocket 연결 해제
    this.disconnectStomp()
  },
  methods: {
    async method2() {

      location.href = '/api/method2';
      // location.href = 'http://localhost:8000/api/method2'
    },
    async method3() {
      const instance = axios.create()
      // 이미지를 불러오는 API 호출
      const response = await instance({
        url: '/api/method3',   // 이미지 API 경로
        method: 'GET',
        responseType: 'blob', // blob 형식으로 응답 받기
      })
      const url = URL.createObjectURL(response.data);
      // 이미지 URL을 data에 저장
      this.imageUrl = url;
    },
    // STOMP 연결
    connectStomp() {
      // 1) SockJS 인스턴스 생성 (서버 Endpoint 예: /ws-stomp)
      const socket = new SockJS('http://localhost:8000/ws/chat/message')

      // 2) SockJS 위에 STOMP를 얹어서 사용
      this.stompClient = Stomp.over(socket)

      // 3) STOMP 서버 연결 시도
      this.stompClient.connect({}, () => {
        this.isConnected = true
        console.log('STOMP Connected')

        // 4) 서버가 메시지를 브로드캐스트하는 경로(subscribe) 구독
        //    (예: /sub/chat/room/room1)
        this.stompClient.subscribe('/sub/chat/room/room1', (message) => {
          const payload = JSON.parse(message.body)
          this.messages.push(payload)
        })
      }, (error) => {
        console.error('STOMP error:', error)
      })
    },

    // STOMP 연결 해제
    disconnectStomp() {
      if (this.stompClient && this.stompClient.connected) {
        this.stompClient.disconnect()
        this.isConnected = false
        console.log('STOMP Disconnected')
      }
    },

    // 메시지 전송
    sendChat() {
      if (this.stompClient && this.stompClient.connected) {
        // 서버에 보낼 데이터
        const data = {
          roomId: 'room1',
          sender: this.sender,
          message: this.content
        }
        // 5) STOMP 발행 (예: /pub/chat/message)
        this.stompClient.send('/pub/chat/message', {}, JSON.stringify(data))
        this.content = ''
      }
    }
  }
}
</script>

<style scoped>
/* 필요하면 스타일 추가 */
</style>
