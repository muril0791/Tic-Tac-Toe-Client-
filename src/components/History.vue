<template>
    <div class="history">
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
    name: 'History',
    props: {
        history: Array
    }
};
</script>

<style scoped>
.history ul {
    max-height: 200px;
    overflow-y: scroll;
    border: 1px solid #ddd;
    padding: 10px;
    background-color: #f9f9f9;
}
</style>
