<template>
  <div id="app">
    <h1>Tic Tac Toe</h1>
    <header v-if="username">
      <button @click="toggleChat">Chat</button>
      <button @click="toggleHistory">History</button>
    </header>
    <Login v-if="!username" @login="handleLogin" />
    <div v-else>
      <div v-if="!currentRoom" class="rooms">
        <h2>Rooms</h2>
        <div v-for="(room, name) in rooms" :key="name" class="room">
          <span>{{ name }}</span>
          <button @click="joinRoom(name)">Join</button>
        </div>
        <input v-model="newRoomName" placeholder="New Room Name" />
        <button @click="createRoom">Create Room</button>
      </div>
      <div v-else class="game">
        <game-board :board="board" @move="makeMove" v-if="gameStarted"></game-board>
        <p v-else>Aguardando jogadores...</p>
        <p v-if="countdown">{{ countdown }}</p>
        <p v-if="status">{{ status }}</p>
        <p v-if="turn">{{ turn === socket.id ? 'Sua vez!' : 'Vez do oponente' }}</p>
        <button @click="leaveRoom">Leave Room</button>
        <Chat v-if="showChat" :messages="messages" @sendMessage="sendMessage" />
        <History v-if="showHistory" :history="history" />
      </div>
    </div>
    <Modal v-if="showModal" @close="closeModal">
      <h3>{{ modalTitle }}</h3>
      <p>{{ modalMessage }}</p>
      <button @click="playAgain">Play Again</button>
      <button @click="leaveRoom">Leave Room</button>
    </Modal>
  </div>
</template>

<script>
import io from 'socket.io-client';
import GameBoard from './components/GameBoard.vue';
import Modal from './components/Modal.vue';
import Chat from './components/Chat.vue';
import History from './components/History.vue';
import Login from './components/Login.vue';

export default {
  name: 'App',
  components: {
    GameBoard,
    Modal,
    Chat,
    History,
    Login,
  },
  data() {
    return {
      socket: null,
      username: '',
      rooms: {},
      currentRoom: null,
      board: Array(9).fill(null),
      status: '',
      gameStarted: false,
      countdown: null,
      newRoomName: '',
      loginError: '',
      showModal: false,
      modalTitle: '',
      modalMessage: '',
      turn: null,
      messages: [],
      newMessage: '',
      history: [],
      showChat: false,
      showHistory: false,
    };
  },
  methods: {
    handleLogin(username) {
      this.username = username;
      this.socket.emit('login', username);
    },
    createRoom() {
      if (this.newRoomName) {
        this.socket.emit('create_room', this.newRoomName);
        this.newRoomName = '';
      }
    },
    joinRoom(roomName) {
      this.socket.emit('join_room', roomName);
      this.currentRoom = roomName;
    },
    leaveRoom() {
      this.socket.emit('leave_room', this.currentRoom);
      this.currentRoom = null;
      this.board = Array(9).fill(null);
      this.status = '';
      this.gameStarted = false;
      this.countdown = null;
      this.turn = null;
      this.messages = [];
      this.history = [];
    },
    makeMove(index) {
      if (this.gameStarted && this.board[index] === null && this.turn === this.socket.id) {
        this.socket.emit('move', { roomName: this.currentRoom, index });
      }
    },
    showEndGameOptions(result) {
      this.showModal = true;
      if (result.winner) {
        if (result.winner === this.socket.id) {
          this.modalTitle = 'Você Ganhou!';
          this.modalMessage = 'Parabéns!';
        } else {
          this.modalTitle = 'Você Perdeu!';
          this.modalMessage = 'Tente novamente!';
        }
      } else {
        this.modalTitle = 'Empate!';
        this.modalMessage = 'Ninguém venceu.';
      }
    },
    playAgain() {
      this.socket.emit('play_again', this.currentRoom);
      this.showModal = false;
    },
    closeModal() {
      this.showModal = false;
    },
    sendMessage(message) {
      if (message.trim()) {
        this.socket.emit('send_message', { roomName: this.currentRoom, message });
        this.newMessage = '';
      }
    },
    toggleChat() {
      this.showChat = !this.showChat;
    },
    toggleHistory() {
      this.showHistory = !this.showHistory;
    },
  },
  created() {
    const serverUrl = import.meta.env.VITE_SERVER_URL;
    this.socket = io(serverUrl, {
      transports: ['websocket', 'polling']
    });

    this.socket.on('connect', () => {
      console.log('Connected to server');
    });

    this.socket.on('logged_in', (data) => {
      this.rooms = data.rooms;
      this.loginError = '';
      console.log('Logged in:', data);
    });

    this.socket.on('login_error', (message) => {
      this.loginError = message;
      console.log('Login error:', message);
    });

    this.socket.on('room_list', (rooms) => {
      this.rooms = rooms;
      console.log('Room list:', rooms);
    });

    this.socket.on('room_update', (room) => {
      if (this.currentRoom === room.name) {
        this.board = room.board;
        this.status = '';
        this.messages = room.messages;
        this.history = room.history;
        console.log('Room update:', room);
      }
    });

    this.socket.on('countdown', (count) => {
      this.countdown = count;
      if (count === 0) {
        this.gameStarted = true;
        this.countdown = null;
      }
      console.log('Countdown:', count);
    });

    this.socket.on('game_start', ({ startingPlayer }) => {
      this.status = `Player ${startingPlayer === this.socket.id ? 'you' : 'opponent'} starts!`;
      this.turn = startingPlayer;
      console.log('Game start. Starting player:', startingPlayer);
    });

    this.socket.on('board_update', (board) => {
      this.board = board;
      console.log('Board update:', board);
    });

    this.socket.on('turn_update', (currentPlayer) => {
      this.turn = currentPlayer;
      console.log('Turn update. Current player:', currentPlayer);
    });

    this.socket.on('game_end', (result) => {
      this.gameStarted = false;
      this.showEndGameOptions(result);
      console.log('Game end. Result:', result);
    });

    this.socket.on('reset_game', () => {
      this.gameStarted = false;
      this.board = Array(9).fill(null);
      this.status = '';
      this.turn = null;
      console.log('Game reset');
    });

    this.socket.on('wait_for_play_again', () => {
      this.status = 'Waiting for the other player to play again...';
      console.log('Waiting for play again');
    });

    this.socket.on('message_update', (messages) => {
      this.messages = messages;
      console.log('Message update:', messages);
    });

    this.socket.on('disconnect', () => {
      this.status = 'Disconnected from server.';
      this.gameStarted = false;
      console.log('Disconnected from server');
    });
  },
};
</script>

<style>
#app {
  text-align: center;
  margin-top: 50px;
  font-family: Arial, sans-serif;
}

header {
  display: flex;
  justify-content: flex-end;
  padding: 10px;
}

button {
  margin-left: 10px;
}

.rooms,
.game {
  margin-top: 20px;
}

input,
button {
  padding: 10px;
  margin: 10px;
  font-size: 16px;
}

button {
  cursor: pointer;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
}

button:hover {
  background-color: #45a049;
}

.board {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  gap: 5px;
  margin: 20px auto;
}

.cell {
  width: 100px;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f0f0f0;
  font-size: 2em;
  cursor: pointer;
  border: 1px solid #ddd;
  border-radius: 5px;
}

.cell:hover {
  background-color: #ddd;
}

.chat,
.history {
  margin-top: 20px;
  text-align: left;
  margin-left: 30%;
  margin-right: 30%;
}

.messages,
.history ul {
  max-height: 200px;
  overflow-y: scroll;
  border: 1px solid #ddd;
  padding: 10px;
  background-color: #f9f9f9;
}

.message {
  margin: 5px 0;
}

.error {
  color: red;
}
</style>
