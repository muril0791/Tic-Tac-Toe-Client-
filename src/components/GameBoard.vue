<template>
    <div class="board">
        <div class="cell" v-for="(cell, index) in board" :key="index" @click="handleClick(index)">
            {{ cell }}
        </div>
    </div>
</template>

<script>
export default {
    name: 'GameBoard',
    props: {
        board: Array,
    },
    methods: {
        handleClick(index) {
            this.$emit('move', index);
        },
    },
};
</script>

<style>
.board {
    display: grid;
    grid-template-columns: repeat(3, 100px);
    gap: 5px;
    margin: auto;
    max-width: 315px;
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
    color: black;
}

.cell:hover {
    background-color: #ddd;
}
</style>
