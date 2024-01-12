<script setup>
import { ref, } from 'vue'
const emit = defineEmits(['isLock', 'nowChangeLrcRow', 'newPushLrcRow'])

const changeLock = () => {
    emit('isLock')
}

const nowChangeLrcRow = () => {
    emit('nowChangeLrcRow')
}

const newPushLrcRow = () => {
    emit('newPushLrcRow')
}


</script>

<template>
    <div>
        <div class="main">
            <button class="iconfont" @click="newPushLrcRow">&#xeb78;</button>
            <button class="iconfont" @click="nowChangeLrcRow">&#xe7c8;</button>
            <button class="iconfont" @click="changeLock">&#xe642;</button>
            <!-- <button class="iconfont" @click="changeLock">&#xe642;</button> -->
        </div>
    </div>
</template>

<style scoped>
.main {
    z-index: 3;
    position: fixed;
    top: 73%;
    left: 10vw;
    display: flex;

    justify-content: center;
    /* 水平方向居中 */
    align-items: center;
    /* 垂直方向居中 */

    border: 1px solid black;
    border-radius: 4px;
    background-color: white;

    height: 50px;
    width: auto;
}

button {
    border: 0;
    padding: 10px;
    background-color: #00000000;
    margin: 0 5px 0;
}

@font-face {
    font-family: "iconfont";
    /* Project id 4376533 */
    src: url('src/assets/font_icon/iconfont.woff2?t=1703485826707') format('woff2'),
        url('src/assets/font_icon/iconfont.woff?t=1703485826707') format('woff'),
        url('src/assets/font_icon/iconfont.ttf?t=1703485826707') format('truetype');
}

.iconfont.openfile {
    font-size: 25px;
}

.iconfont {
    font-family: "iconfont" !important;
    font-size: 25px;
    font-style: normal;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;

    -moz-user-select: none;
    /*火狐*/
    -webkit-user-select: none;
    /*webkit浏览器*/
    -ms-user-select: none;
    /*IE10*/
    -khtml-user-select: none;
    /*早期浏览器*/
    user-select: none;
}
</style>