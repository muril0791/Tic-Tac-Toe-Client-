<template>
    <div class="login">
        <input v-model="username" placeholder="Enter your username (Player1 or Player2)" />
        <button @click="login">Login</button>
        <p v-if="loginError" class="error">{{ loginError }}</p>
    </div>
</template>

<script>
export default {
    name: 'Login',
    data() {
        return {
            username: '',
            loginError: ''
        };
    },
    methods: {
        login() {
            if (this.username.trim()) {
                this.$emit('login', this.username);
                this.loginError = '';
            } else {
                this.loginError = 'Username is required';
            }
        }
    }
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

.error {
    color: red;
}
</style>
