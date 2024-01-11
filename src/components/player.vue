<script setup>
import progressBar from './player/progress_bar.vue'
import control from './player/control.vue'
import bus from '@/components/plugin/bus.js'

import { nextTick, ref, watch, provide } from 'vue'


const isFile = ref(false)

const fileInputDom = ref(null)
const audioElement = ref(null)
const audioFileName = ref("")
const audioDuration = ref(null)
const audioCurrentTime = ref(null)
const audioFormatDuration = ref("00:00.00")
const audioFormatCurrent = ref("00:00.00")
const audioToPercentage = ref(0)
const audioFile = ref(null)
const audioVolume = ref(0.45)
const audioSpeed = ref(1.0)

const isPlay = ref(false)

const handleFileChange = (event) => {
    audioFile.value = event.target.files[0]
    console.log(audioFile.value)
    if (audioFile.value) {
        isFile.value = true
        audioFileName.value = audioFile.value.name
        const audioURL = URL.createObjectURL(audioFile.value)
        console.log(`${audioURL}`);
        audioElement.value.src = audioURL
        nextTick(() => {
            console.log(audioElement.value);
            audioDuration.value = audioElement.value.duration
            audioElement.value.volume = audioVolume.value
            audioElement.value.playbackRate = audioSpeed.value
        })

    }
}

const nextInputFile = () => {
    fileInputDom.value.click()
    nextTick(() => {
        pause()
        setProgress(0)
    })
}

const getDuration = () => {
    audioDuration.value = audioElement.value.duration
    audioFormatDuration.value = formatTime(audioDuration.value)

}

const getCurrentTime = () => {
    audioCurrentTime.value = parseFloat(audioElement.value.currentTime)
}

const play = () => {
    isPlay.value = true
    audioElement.value.play().catch(error => {
        console.log(audioFile.value)
        console.error(error)
        audioElement.value.src = URL.createObjectURL(audioFile.value)
        audioElement.value.play();
    })
}

const pause = () => {
    isPlay.value = false
    audioElement.value.pause()
}

const setVolume = (volume) => {
    if (volume >= 0 && volume <= 1) {
        audioVolume.value = volume
        audioElement.value.volume = volume;

    }
}

const setProgress = (percent) => {
    if (percent >= 0 && percent <= 100) {
        const duration = audioElement.value.duration;
        const newTime = (percent / 100) * duration;
        audioElement.value.currentTime = newTime;
        audioCurrentTime.value = audioElement.value.currentTime
    }
}

const setCurrentTime = (seconds) => {
    if (seconds >= 0 && seconds <= audioDuration.value && !isNaN(seconds)) {
        audioElement.value.currentTime = seconds
        audioCurrentTime.value = audioElement.value.currentTime
    }
}

const setPlaybackRate = (speed) => {
    if (speed >= 0.1 && speed <= 3) {
        audioElement.value.playbackRate = speed;
        audioSpeed.value = audioElement.value.playbackRate
    }
}

function formatTime(seconds) {
    // 计算分钟、秒数和百分之一秒
    const minutes = Math.floor(seconds / 60);
    const remainingSeconds = Math.floor(seconds % 60);
    const tenths = Math.floor((seconds * 100) % 100);

    // 格式化成字符串
    const formattedMinutes = minutes >= 100 ? '100' : (minutes < 10 ? '0' : '') + minutes;
    const formattedSeconds = (remainingSeconds < 10 ? '0' : '') + remainingSeconds;
    const formattedTenths = (tenths < 10 ? '0' : '') + tenths;

    // 返回格式化后的字符串
    return `${formattedMinutes}:${formattedSeconds}.${formattedTenths}`;
}

const endAgain = () => {
    console.log('end');
    pause()

}

/* provide('CurrentTime', audioCurrentTime)
provide('FormatCurrent', audioFormatCurrent) */

watch(
    audioCurrentTime,
    () => {

        audioFormatCurrent.value = formatTime(audioCurrentTime.value)
        audioToPercentage.value = ((audioCurrentTime.value / audioDuration.value) * 100).toFixed(2)
        bus.emit('CurrentTime', audioCurrentTime)

        bus.emit('FormatCurrent', audioFormatCurrent)
    }
)



</script>

<template>
    <div class="main-smooth " :class="{ 'mainNoFile': !isFile, 'main': isFile }">
        <div>
            <audio ref="audioElement" @timeupdate="getCurrentTime" @canplay="getDuration" @ended="endAgain">
            </audio>
        </div>


        <div v-if="isFile">
            <div>
                <div class="blank"></div>
                <progressBar :isPlay="isPlay" :fileName="audioFileName" :audioFormatDuration="audioFormatDuration"
                    :audioFormatCurrent="audioFormatCurrent" :changePosition="audioToPercentage"
                    @progresssonPosition="setProgress" @progresssonplay="play" @progresssonpause="pause">
                </progressBar>
            </div>
            <div class="control">
                <control :isPlay="isPlay" :currentTime="audioCurrentTime" :audioVolume="audioVolume"
                    :audioSpeed="audioSpeed" @sonplay="play" @sonpause="pause" @sonSetCurrentTime="setCurrentTime"
                    @sonInputFile="nextInputFile" @sonSetVolume="setVolume" @sonSetSpeed="setPlaybackRate">
                </control>
            </div>
        </div>


        <label ref="fileInputDom" v-show="!isFile" for="file-input" class=" iconfont openfile">&#xe700;打开音频文件
            <input id="file-input" type="file" accept="audio/*" @change="handleFileChange">
        </label>
    </div>
</template>


<style scoped>
.iconfont.openfile {
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

/* .file-label {
    padding: 6px 25px;
    background: #00bfff;
    border-radius: 4px;
    color: white;
    cursor: pointer;
} */

#file-input {
    display: none
}

.main {
    z-index: 3;
    position: fixed;
    top: 80%;
    left: 10vw;
    display: flex;
    flex-direction: column;
    /* 将容器的主轴方向设置为垂直方向 */
    justify-content: center;
    /* 水平方向居中 */
    align-items: center;
    /* 垂直方向居中 */

    border: 1px solid black;
    border-radius: 4px;
    background-color: white;

    height: 100px;
    width: 80vw;

}

.mainNoFile {
    position: fixed;
    top: 90%;
    left: 35vw;
    display: flex;
    flex-direction: column;
    /* 将容器的主轴方向设置为垂直方向 */
    justify-content: center;
    /* 水平方向居中 */
    align-items: center;
    /* 垂直方向居中 */

    border: 1px solid black;
    border-radius: 4px;
    background-color: white;

    height: 60px;
    width: 30vw;

}

.main-smooth {
    transition: 0.25s ease;
}


.blank {
    height: 20px;
    width: 10px;
}

.control {
    position: relative;
    top: -0.3vh;
}

.prohibit-selection {
    -moz-user-select: none;
    -webkit-user-select: none;
    -ms-user-select: none;
    -khtml-user-select: none;
    user-select: none;
}
</style>