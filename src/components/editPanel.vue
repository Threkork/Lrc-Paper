<script setup>
import tool from '@/components/editPanel/tool.vue'

import { ref, watch, computed, inject, onMounted } from 'vue'
import bus from '@/components/plugin/bus.js'

const lrcListDom = ref([])

const isLockCurrent = ref(true)

const audioCurrentTime = ref(null)
const audioFormatCurrent = ref('00:00.00')

onMounted(() => {
    bus.on('CurrentTime', e => {
        audioCurrentTime.value = e.value
    })
    bus.on('FormatCurrent', e => {
        audioFormatCurrent.value = e.value
    })
})

const InputlrcStr = ref('')
const lrcArr = ref([])

const saveLrcToFile = () => {
    let lrcContent = "";

    lrcArr.value.forEach((item) => {
        /* const time = formatTime(item.type); */
        const line = `[${item.type}] ${item.information}\n`;
        lrcContent += line;
    });

    const blob = new Blob([lrcContent], { type: "text/plain" });

    const a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = "userInput.lrc";
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
};

const formatTime = (time) => {
    const minutes = Math.floor(time / 60);
    const seconds = (time % 60).toFixed(2);
    return `${String(minutes).padStart(2, '0')}:${seconds.padStart(5, '0')}`;
};

function createLRCrow(type = '', information = '') {
    const lrcItem = {
        type: type,
        information: information,
        showAddBtn: false
    };
    return lrcItem
}

function parseLRC(lrcString) {
    const lrcArray = [];

    const lines = lrcString.split('\n');
    const lrcRegex = /\[(\d+:\d+\.\d+)\]\s*(.+)/;

    for (const line of lines) {
        const match = line.match(lrcRegex);
        if (match) {
            const type = match[1];
            const information = match[2];

            const lrcItem = createLRCrow(type, information)

            lrcArray.push(lrcItem);
        }
    }

    return lrcArray;
}

const handleInputLrc = (event) => {
    const fileInput = event.target;
    const file = fileInput.files[0];

    if (file) {
        const reader = new FileReader();

        reader.onload = function (e) {
            InputlrcStr.value = e.target.result
            lrcArr.value = parseLRC(InputlrcStr.value)
        };

        reader.readAsText(file)
    }
}

const showAddBtn = (index) => {
    lrcArr.value[index].showAddBtn = true
}
const hideAddBtn = (index) => {
    lrcArr.value[index].showAddBtn = false
}


const addNewRow = (index) => {
    const newLrcItem = createLRCrow()
    lrcArr.value.splice(index, 0, newLrcItem)
}
const deleteRow = (index) => {
    lrcArr.value.splice(index, 1)
}

const nowAddRow = () => {
    const newLrcItem = createLRCrow(audioFormatCurrent.value)
    lrcArr.value.push(newLrcItem)
}

const anchorPosition = (index) => {
    const element = lrcListDom.value[index]
    element.scrollIntoView({
        block: 'center',
        behavior: 'smooth'
    })
}

const changeLockCurrentBool = () => {
    isLockCurrent.value = !isLockCurrent.value
}

const lockCurrentIndex = ref(0)

watch(
    audioFormatCurrent,
    (newVal, oldVal) => {

        if (lrcArr.value.length === 0) {
            return
        }

        const newTime = parseInt(newVal.replace(/[:.]/g, ""), 10)
        const oldTime = parseInt(oldVal.replace(/[:.]/g, ""), 10)

        const findIndex = () => {

            for (let index = 0; index <= lrcArr.value.length; index++) {
                console.log(index);
                if (lrcArr.value.length === index) {

                    if (lrcArr.value.length === 0) {
                        return 0
                    } else {
                        return index - 1
                    }

                }

                const element = lrcArr.value[index];
                if (newTime > parseInt(element.type.replace(/[:.]/g, ""), 10)) {
                    continue
                }

                if (element.type === '') {
                    continue
                }

                if (index === 0) {
                    return index
                } else {
                    return index - 1
                }

            }
        }


        //if (newTime - oldTime >= 100 || newTime < oldTime) { //性能模式 配合下面的使用，如果新的时间和旧时间差1秒，或者新时间小于旧时间，就会触发findIndex，否则触发逐层自增
        const findIndex_ = findIndex()
        lockCurrentIndex.value = findIndex_
        if (!isLockCurrent.value) {
            anchorPosition(lockCurrentIndex.value)
        }
        return
        //}


        /* 
        //性能模式
        const a = (index) => {
            if (lrcArr.value.length === 0) {
                return null
            }
            if (lrcArr.value.length === index + 1) {
                return null
            }
            if (lrcArr.value[index].type === '') {
                return a(index + 1)
            }

            if (newTime < parseInt(lrcArr.value[index + 1].type.replace(/[:.]/g, ""), 10)) {
                return null
            }

            return index + 1

        }
        const some = a(lockCurrentIndex.value)
        if (some !== null) {
            lockCurrentIndex.value = some
            anchorPosition(lockCurrentIndex.value)
        } */
    })

</script>

<template>
    <tool @isLock="changeLockCurrentBool"></tool>
    <div class="main">
        <input type="file" accept=".lrc" @change="handleInputLrc">
        <button @click="saveLrcToFile">保存到文件</button>
        <div>
            <div class="space1"></div>
            <div v-for="(item, index) in   lrcArr  " :key="index" ref="lrcListDom" class="center lrc-row"
                @mouseover="showAddBtn(index)" @mouseleave="hideAddBtn(index)">

                <input class=" inputBox inputType" v-model="item.type" :class="{ 'inputNull': item.type === '' }" />
                <textarea class="inputBox inputInformation" v-model="item.information" rows="10"
                    :class="{ 'inputNull': item.information === '' }"></textarea>
                <div class="center spliceArrBox">
                    <div v-show="item.showAddBtn" class="spliceArr mouse-pointer prohibit-selection"
                        @click="addNewRow(index + 1)">增加</div>
                    <div v-show="item.showAddBtn" class="spliceArr mouse-pointer prohibit-selection"
                        @click="deleteRow(index)">
                        删除</div>
                </div>
                <div :class="{ 'lockCurrent': index === lockCurrentIndex }"></div>

            </div>
            <div class="space1"></div>
        </div>
    </div>
    <div @click="nowAddRow">此刻</div>
    <div @click="changeLockCurrentBool">转跳</div>
    <div class="foot"></div>
</template>

<style scoped>
.main {
    margin: 2% 20% 0;
    min-height: 900px;
    display: flex;
    flex-direction: column;
    /* 将容器的主轴方向设置为垂直方向 */

    align-items: center;
    /* 垂直方向居中 */
    border: 1px solid black;
    border-radius: 4px;
    background-color: white;
}

.center {
    display: flex;
    justify-content: center;
    /* 水平方向居中 */
    align-items: center;
    /* 垂直方向居中 */
}

.lrc-row {
    position: relative;
    width: 59vw;
}

.lockCurrent {
    position: absolute;
    width: 40vw;
    top: 35px;
    border-bottom-style: solid;
    border-color: black;
    border-width: 1px;
    z-index: 3;
}

.inputBox {
    font-size: 18px;
    height: 30px;
    margin-top: 5px;
    margin-bottom: 5px;
}



.inputType {
    width: 80px;
    margin-right: 20px;
    border: none;
}

.inputInformation {
    position: relative;
    resize: none;
    height: 30px;
    width: 30vw;
    font-size: 18px;
    color: black;
    border: none;
    font-family: ARIAL, "Microsoft Yahei", "微软雅黑";

}

.inputNull {
    border-bottom: 1px solid rgb(0, 0, 0);
}

.spliceArrBox {
    position: absolute;
    left: 90%;
}

.spliceArr {
    margin: 3px;
    text-align: center;
    background-color: white;
}

.addArr {
    position: fixed;
    top: 70%;
    right: 20%;
    height: 50px;
    width: 50px;

    text-align: center;
    text-align-last: center;
    justify-content: center;
    align-items: center;

    border: 1px solid black;
    border-radius: 50%;
    background-color: white;
}

.head {
    height: 100px;
}

.foot {
    height: 300px;
}

.mouse-pointer {
    cursor: pointer;
}


.prohibit-selection {
    -moz-user-select: none;
    -webkit-user-select: none;
    -ms-user-select: none;
    -khtml-user-select: none;
    user-select: none;
}

.space1 {
    height: 40px;
}
</style>