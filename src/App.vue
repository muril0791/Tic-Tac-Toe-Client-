<template>
  <div id="app">
    <h1>Tic Tac Toe</h1>
    <header v-if="username">

      <div v-if="currentRoom" class="toggle-buttons">
        <button class="toggle-button" @click="toggleChat">Chat</button>
        <button class="toggle-button" @click="toggleHistory">History</button>
      </div>
    </header>
    <login v-if="!username" @login="handleLogin" :loginError="loginError"></login>
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
      </div>
    </div>
    <modal v-if="showModal" @close="closeModal">
      <h3>{{ modalTitle }}</h3>
      <p>{{ modalMessage }}</p>
      <button @click="playAgain">Play Again</button>
      <button @click="leaveRoom">Leave Room</button>
    </modal>
    <chat v-if="showChat" :messages="messages" @send-message="sendMessage" @close="toggleChat"></chat>
    <history v-if="showHistory" :history="history" @close="toggleHistory"></history>
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
    this.socket = io(serverUrl);

    this.socket.on('logged_in', (data) => {
      this.rooms = data.rooms;
      this.loginError = '';
    });

    this.socket.on('login_error', (message) => {
      this.loginError = message;
    });

    this.socket.on('room_list', (rooms) => {
      this.rooms = rooms;
    });

    this.socket.on('room_update', (room) => {
      if (this.currentRoom === room.name) {
        this.board = room.board;
        this.status = '';
        this.messages = room.messages;
        this.history = room.history;
      }
    });

    this.socket.on('countdown', (count) => {
      this.countdown = count;
      if (count === 0) {
        this.gameStarted = true;
        this.countdown = null;
      }
    });

    this.socket.on('game_start', ({ startingPlayer }) => {
      this.status = `Player ${startingPlayer === this.socket.id ? 'you' : 'opponent'} starts!`;
      this.turn = startingPlayer;
    });

    this.socket.on('board_update', (board) => {
      this.board = board;
    });

    this.socket.on('turn_update', (currentPlayer) => {
      this.turn = currentPlayer;
    });

    this.socket.on('game_end', (result) => {
      this.gameStarted = false;
      this.showEndGameOptions(result);
    });

    this.socket.on('reset_game', () => {
      this.gameStarted = false;
      this.board = Array(9).fill(null);
      this.status = '';
      this.turn = null;
    });

    this.socket.on('wait_for_play_again', () => {
      this.status = 'Waiting for the other player to play again...';
    });

    this.socket.on('message_update', (messages) => {
      this.messages = messages;
    });

    this.socket.on('disconnect', () => {
      this.status = 'Disconnected from server.';
      this.gameStarted = false;
    });
  },
};
</script>

<style>
#app {
  text-align: center;
  font-family: Arial, sans-serif;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background-color: #282c34;
  color: white;
}

.toggle-buttons {
  display: flex;
  gap: 10px;
}

.toggle-button {
  background: none;
  border: none;
  color: white;
  font-size: 16px;
  cursor: pointer;
}

.login,
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
  position: fixed;
  top: 0;
  right: 0;
  width: 300px;
  height: 100%;
  background: #20232a;
  color: white;
  display: flex;
  flex-direction: column;
  padding: 20px;
}

.messages,
.history ul {
  flex-grow: 1;
  overflow-y: auto;
  margin-bottom: 10px;
}

.message {
  margin-bottom: 10px;
}

input[type="text"] {
  padding: 10px;
  margin-bottom: 10px;
  font-size: 16px;
  width: 100%;
}

button[type="submit"] {
  cursor: pointer;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  width: 100%;
  padding: 10px;
  font-size: 16px;
}

button[type="submit"]:hover {
  background-color: #45a049;
}

.error {
  color: red;
}
</style>
