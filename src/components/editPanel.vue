<script setup>
import tool from '@/components/editPanel/tool.vue'

import { ref, watch, computed, inject, onMounted } from 'vue'
import bus from '@/components/plugin/bus.js'

const fileName = ref('Music-Name')
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
const lrcArr = ref([
    { "type": "00:00.00", "information": "鼠标移上来，最左侧按钮点击可选中", "showAddBtn": false },
    { "type": "00:00.00", "information": "选中后，点击工具栏铅笔，可填入此刻", "showAddBtn": false },
    { "type": "00:00.00", "information": "完成后，工具栏的锁，可以锁定预览", "showAddBtn": false },
    { "type": "00:00.00", "information": "左上角可打开lrc文件，右上角可下载保存", "showAddBtn": false },
    { "type": "00:00.00", "information": "", "showAddBtn": false },
    { "type": "00:00.00", "information": "", "showAddBtn": false }
])

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
    if (fileName.value === '') {
        a.download = `LrcPaper.lrc`;
    } else {
        a.download = `${fileName.value}.lrc`;
    }

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
    console.log(file);
    if (file) {
        const reader = new FileReader();

        reader.onload = function (e) {
            InputlrcStr.value = e.target.result
            lrcArr.value = parseLRC(InputlrcStr.value)
        };

        reader.readAsText(file)
        fileName.value = file.name.substring(0, file.name.lastIndexOf("."))
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
    anchorPosition(lrcArr.value.length - 2)
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

const selectedCurrentIndex = ref(-1)

const changeSelectedCurrentIndex = (index) => {
    if (index === selectedCurrentIndex.value) {
        selectedCurrentIndex.value = -1
        return
    }
    selectedCurrentIndex.value = index
}

const nowChangeLrcRow = () => {
    if (selectedCurrentIndex.value === -1) {
        return
    }
    lrcArr.value[selectedCurrentIndex.value].type = audioFormatCurrent.value
    selectedCurrentIndex.value += 1
    if (selectedCurrentIndex.value == lrcArr.value.length) {
        selectedCurrentIndex.value = -1
    }
    anchorPosition(selectedCurrentIndex.value)
}



</script>

<template>
    <tool @isLock="changeLockCurrentBool" @nowChangeLrcRow="nowChangeLrcRow" @newPushLrcRow="nowAddRow"></tool>

    <div class="main">
        <div class="file-box">
            <label for="lrc-file-input" class="iconfont openfile">&#xe700;
                <input v-show="false" id="lrc-file-input" type="file" accept=".lrc" @change="handleInputLrc">
            </label>

            <button class="iconfont" @click="saveLrcToFile">&#xe607;</button>
        </div>
        <div>
            <input class=" inputBox inputName" v-model="fileName" :class="{ 'inputNull': fileName === '' }" />
        </div>
        <div>
            <div class="space1"></div>
            <div v-for="(item, index) in   lrcArr  " :key="index" ref="lrcListDom" class="center lrc-row"
                @mouseover="showAddBtn(index)" @mouseleave="hideAddBtn(index)">
                <button v-show="item.showAddBtn" class="button-lrc-row" @click="changeSelectedCurrentIndex(index)"></button>
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
                <div v-show="index === lockCurrentIndex && -1 === selectedCurrentIndex" class="lockCurrent">
                </div>
                <div v-show="index === selectedCurrentIndex && selectedCurrentIndex !== -1" class="lockCurrent-lrc-row">
                </div>


            </div>
            <div class="space1"></div>
        </div>
    </div>
    <div class="foot"></div>
</template>

<style scoped>
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

.main {
    margin: 2% 20% 0;
    min-height: 900px;
    display: flex;
    flex-direction: column;
    align-items: center;

    border: 1px solid black;
    border-radius: 4px;
    background-color: white;
}

.center {
    display: flex;
    justify-content: center;
    align-items: center;
}

.lrc-row {
    position: relative;
    width: 59vw;
}

.lockCurrent {
    position: absolute;
    width: 40vw;
    top: 34px;
    border-bottom-style: solid;
    border-color: black;
    border-width: 1px;
    z-index: 3;
}

.lockCurrent-lrc-row {
    position: absolute;
    width: 40vw;
    top: 34px;
    border-bottom-style: dashed;
    border-color: black;
    border-width: 1px;
    z-index: 3;
}

.button-lrc-row {
    position: absolute;
    left: 13%;
    width: 18px;
    height: 18px;
    border-width: 1px;
    border-radius: 4px;
    background-color: white;
    margin-right: 10px;
}

.inputBox {
    font-size: 18px;
    height: 30px;
    margin-top: 5px;
    margin-bottom: 5px;
}

.inputName {
    position: relative;
    resize: none;
    width: 40vw;
    height: 30px;
    font-size: 22px;
    text-align: center;
    color: black;
    border: none;
    font-family: ARIAL, "Microsoft Yahei", "微软雅黑";
}

.inputType {
    position: relative;
    resize: none;
    width: 80px;
    height: 30px;
    margin-right: 20px;
    font-size: 18px;
    color: black;
    border: none;
    font-family: ARIAL, "Microsoft Yahei", "微软雅黑";
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

.file-box {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
}

.file-box .openfile {
    margin: 15px 15px 5px;
    color: #000000;
    padding: 0 0px 10px;
    font-size: 22.5px;
}

.file-box button {
    margin: 15px 15px 5px;
    color: #000000;
    padding: 0 0px 10px;
    font-size: 27px;
    background-color: white;
    border: none;
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