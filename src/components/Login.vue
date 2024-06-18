<template>
    <div class="login">
        <input v-model="username" placeholder="Enter your username (Player1 or Player2)" />
        <button @click="login">Login</button>
        <p v-if="loginError" class="error">{{ loginError }}</p>
    </div>
</template>

<script>
export default {
    props: ['loginError'],
    data() {
        return {
            username: '',
        };
    },
    methods: {
        login() {
            this.$emit('login', this.username);
        },
    },
};
</script>

<style scoped>
.login {
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

.error {
    color: red;
}
</style>
