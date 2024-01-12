<script setup>
import volumeCardDom from './volumeCard.vue';
import speedCardDom from './speedCard.vue';

import { ref, watch } from 'vue'

const props = defineProps({
    isPlay: Boolean,
    currentTime: Number,
    audioVolume: Number,
    audioSpeed: Number,
    durationTime: Number
})

const emit = defineEmits(['sonplay', 'sonpause', 'sonSetCurrentTime', 'sonInputFile', 'sonSetVolume', 'sonSetSpeed'])

const play = () => {
    emit('sonplay')
}
const pause = () => {
    emit('sonpause')
}

const inputFile = () => {
    closeAll()
    emit('sonInputFile')
}

const setCurrent = (seconds) => {
    let newTime = props.currentTime
    if (props.currentTime < -seconds) {
        newTime = 0
    } else if (props.currentTime + seconds > props.durationTime) {
        newTime = props.durationTime
    } else {
        newTime = newTime + seconds
    }

    emit('sonSetCurrentTime', newTime)
}

const pushVolume = (volume) => {
    emit('sonSetVolume', volume)
}
const pushSpeed = (speed) => {
    emit('sonSetSpeed', speed)
}

const wall = ref(false)


const inputFileCard = ref(false)
const openInputFileCard = () => {
    inputFileCard.value = true
    wall.value = true
}

const volumeCard = ref(false)
const openVolumeCard = () => {
    volumeCard.value = true
    wall.value = true
}

const speedCard = ref(false)
const openSpeedCard = () => {
    speedCard.value = true
    wall.value = true
}

const closeAll = () => {
    inputFileCard.value = false
    volumeCard.value = false
    speedCard.value = false
    wall.value = false
}
</script>

<template>
    <div class="wall" v-if="wall" @click="closeAll"></div>

    <div class="box">

        <div class="left-element center">
            <i class="iconfont inputFile mouse-pointer" @click="openInputFileCard">&#xe700;</i>
            <div class="inputFileCard center center-column " v-show="inputFileCard">
                <div>
                    <i class="iconfont prohibit-selection">&#xe700;更换音频文件</i>
                    <div class="inputFileCardBool">
                        <h5 id="inputFileCardBooltrue" class="left-element mouse-pointer prohibit-selection"
                            @click="inputFile">确定</h5>
                        <h5 id="inputFileCardBoolfalse" class="right-element mouse-pointer prohibit-selection"
                            @click="closeAll">取消</h5>
                    </div>
                </div>
            </div>
        </div>


        <div class="center centered-element">
            <i class="iconfont big left5 mouse-pointer" @click="setCurrent(-5)">&#xe83e;</i>
            <i class="iconfont big left2 mouse-pointer" @click="setCurrent(-2)">&#xe83d;</i>
            <div class="normalbox">
                <i class="iconfont normal mouse-pointer" v-show="!isPlay" @click="play">&#xea6e;</i>
                <i class="iconfont normal mouse-pointer" v-show="isPlay" @click="pause">&#xea6d;</i>
            </div>
            <i class="iconfont big right2 mouse-pointer" @click="setCurrent(2)">&#xe840;</i>
            <i class="iconfont big right5 mouse-pointer" @click="setCurrent(5)">&#xe841;</i>
        </div>

        <div class="right-element">
            <div class="center">
                <div>
                    <div>
                        <i class="iconfont volume mouse-pointer" v-show="audioVolume === 0"
                            @click="openVolumeCard">&#xea0c;</i>
                        <i class="iconfont volume mouse-pointer" v-show="audioVolume > 0 && audioVolume <= 0.35"
                            @click="openVolumeCard">&#xea0e;</i>
                        <i class="iconfont volume mouse-pointer" v-show="audioVolume > 0.35 && audioVolume <= 0.70"
                            @click="openVolumeCard">&#xea0f;</i>
                        <i class="iconfont volume mouse-pointer" v-show="audioVolume > 0.70"
                            @click="openVolumeCard">&#xea0d;</i>
                        <div>
                            <div class="volumeCard center center-column " v-show="volumeCard">
                                <div>
                                    <volumeCardDom :changeVolume="audioVolume" @volumesonPosition="pushVolume">
                                    </volumeCardDom>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div>
                    <i class="iconfont speed mouse-pointer" @click="openSpeedCard">&#xe6b8;</i>
                    <div class="speedCardBox" v-show="speedCard">
                        <speedCardDom class="speedCardSonBox" :speed="audioSpeed" @pushSpeed="pushSpeed"></speedCardDom>
                    </div>
                </div>
            </div>
        </div>

    </div>
</template>

<style scoped>
@font-face {
    font-family: "iconfont";
    /* Project id 4376533 */
    src: url('src/assets/font_icon/iconfont.woff2?t=1703485826707') format('woff2'),
        url('src/assets/font_icon/iconfont.woff?t=1703485826707') format('woff'),
        url('src/assets/font_icon/iconfont.ttf?t=1703485826707') format('truetype');
}

.box {
    display: flex;
    justify-content: space-between;
    align-items: center;

    width: 70vw;
    position: relative;

}

.centered-element {
    position: absolute;
    left: 50%;
    /* 相对于父容器左边的距离为50% */
    transform: translateX(-50%);
    /* 将元素左移自身宽度的一半，使其绝对居中 */
}

.center {
    display: flex;
    justify-content: center;
    /* 水平方向居中 */
    align-items: center;
    /* 垂直方向居中 */
}

.mouse-pointer {
    cursor: pointer;
}

.center-column {
    flex-direction: column;
}

.center-row {
    flex-direction: row;
}

.wall {
    width: 100%;
    height: 100%;
    position: fixed;
    background-color: #ffffff00;
    top: 0;
    left: 0;
    z-index: 4;
}

.speedCardBox {
    position: absolute;
    z-index: 5;
    height: 70px;
    width: 250px;
    background-color: white;
    border: 1px solid #000000;
    border-radius: 4px;

    top: -70px;
    right: 0px;
    font-size: 22.5px;
}

/* .speedCardSonBox {
    position: absolute;
    top: 50px
} */

.volumeCard {
    position: absolute;
    z-index: 5;
    height: 170px;
    width: 55px;
    background-color: white;
    border: 1px solid #000000;
    border-radius: 4px;

    top: -170px;
    right: 18px;
    font-size: 22.5px;
}

.iconfont.inputFile {
    color: #626f7d;
    padding: 0 0px 10px;
    font-size: 22.5px;
}

.inputFileCard {
    position: absolute;
    z-index: 5;
    height: 100px;
    width: 200px;
    background-color: white;
    border: 1px solid #000000;
    border-radius: 4px;

    top: -100px;
    left: -2px;
    font-size: 22.5px;
}


.inputFileCardBool {
    display: flex;
    align-items: center;
    /* 垂直方向居中 */
    justify-content: space-between;
    width: 160px;
}

#inputFileCardBooltrue {
    margin-left: 20px;
}

#inputFileCardBoolfalse {
    margin-right: 20px;
}

.iconfont.big {

    padding: 0 7px 10px;
    font-size: 25px;
}

.iconfont.volume {
    position: relative;
    top: -5.5px;
    padding: 0;
    font-size: 30px;

}

.iconfont.volume:last-child {
    position: relative;
    top: -5.5px;
    padding: 0;
    font-size: 30px;

}

.iconfont.speed {
    color: #000000;
    position: relative;
    top: -7px;
    left: 6.5px;
    padding: 0 0px 0 0px;
    font-size: 27px;
}

.normalbox {
    display: flex;
    /* 将容器的主轴方向设置为垂直方向 */
    justify-content: center;
    /* 水平方向居中 */
    align-items: center;
    /* 垂直方向居中 */
    width: 50px;
}

.iconfont.normal {

    padding: 0 7px 10px;
    font-size: 25px;
}

.iconfont {
    font-family: "iconfont" !important;
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

.prohibit-selection {
    -moz-user-select: none;
    -webkit-user-select: none;
    -ms-user-select: none;
    -khtml-user-select: none;
    user-select: none;
}
</style>