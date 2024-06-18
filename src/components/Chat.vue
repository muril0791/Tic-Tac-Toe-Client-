<template>
    <div class="chat">
        <button class="close-btn" @click="close">×</button>
        <h3>Chat</h3>
        <div class="messages">
            <div v-for="(msg, index) in messages" :key="index" class="message">
                <strong>{{ msg.player }}:</strong> {{ msg.message }}
            </div>
        </div>
        <input v-model="message" placeholder="Type a message" @keyup.enter="sendMessage" />
        <button @click="sendMessage">Send</button>
    </div>
</template>

<script>
export default {
    props: {
        messages: Array,
    },
    data() {
        return {
            message: '',
        };
    },
    methods: {
        sendMessage() {
            if (this.message.trim()) {
                this.$emit('send-message', this.message);
                this.message = '';
            }
        },
        close() {
            this.$emit('close');
        },
    },
};
</script>

<style scoped>
.chat {
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

.close-btn {
    background: none;
    border: none;
    color: white;
    font-size: 24px;
    cursor: pointer;
    align-self: flex-end;
}

.messages {
    flex-grow: 1;
    overflow-y: auto;
    margin-bottom: 10px;
}

.message {
    margin-bottom: 10px;
}

input {
    padding: 10px;
    margin: 10px 0;
    font-size: 16px;
    width: 100%;
}

button {
    cursor: pointer;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 5px;
    width: 100%;
    padding: 10px;
    font-size: 16px;
}

button:hover {
    background-color: #45a049;
}
</style>
