<template>
    <div class="history">
        <button class="close-btn" @click="close">×</button>
        <h3>History</h3>
        <ul>
            <li v-for="(game, index) in history" :key="index">
                Game {{ index + 1 }}: {{ game.winner ? `${game.winner} won` : 'Draw' }}
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    props: {
        history: Array,
    },
    methods: {
        close() {
            this.$emit('close');
        },
    },
};
</script>

<style scoped>
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

.close-btn {
    background: none;
    border: none;
    color: white;
    font-size: 24px;
    cursor: pointer;
    align-self: flex-end;
}

ul {
    list-style: none;
    padding: 0;
}

li {
    margin-bottom: 10px;
}
</style>
