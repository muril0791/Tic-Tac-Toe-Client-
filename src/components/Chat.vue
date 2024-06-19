<template>
    <div class="chat">
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
    name: 'Chat',
    props: {
        messages: Array
    },
    data() {
        return {
            message: ''
        };
    },
    methods: {
        sendMessage() {
            if (this.message.trim()) {
                this.$emit('sendMessage', this.message);
                this.message = '';
            }
        }
    }
};
</script>

<style scoped>
.chat {
    text-align: left;
}

.messages {
    max-height: 200px;
    overflow-y: scroll;
    border: 1px solid #ddd;
    padding: 10px;
    background-color: #f9f9f9;
}

.message {
    margin: 5px 0;
}
</style>
