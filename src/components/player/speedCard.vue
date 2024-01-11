<script setup>
import { nextTick, ref, watch } from 'vue'

const speedInputDom = ref(null)
const openSpeedInput = ref(false)
const speedInput = ref('')
const inputErr = ref(false)

const testSpeed = (speed) => {
    if (speed < 0.1 || speed > 3 || isNaN(speed)) {
        return false
    } else {
        return true
    }
}

const emit = defineEmits(['pushSpeed'])

const pushSpeed = (speed) => {
    emit('pushSpeed', speed)
    openSpeedInput.value = false
}

const openInput = () => {
    openSpeedInput.value = true
    nextTick(() => {
        speedInputDom.value.focus()
        speedInputDom.value.select()
    }

    )
}

const testAndPushSpeed = (speed) => {
    if (testSpeed(speed)) {
        pushSpeed(speed)
    }
}

const removeInput = () => {
    openSpeedInput.value = false
}

const props = defineProps({
    speed: Number,
})

watch(
    speedInput,
    () => {
        if (testSpeed(speedInput.value)) {
            inputErr.value = true
        } else {
            inputErr.value = false
        }
    }
)
watch(
    () => props.speed,
    () => {
        console.log(props.speed);
        speedInput.value = props.speed
    },
    {
        immediate: true
    }
)

</script>

<template>
    <div>
        <div class="speedCardShowBox">
            <div class="container speedListBox">
                <div class="speedList mouse-pointer prohibit-selection" :class="{ 'speedListAndFather': speed === 0.3 }"
                    @click="pushSpeed(0.3)">0.3</div>
                <div class="speedList mouse-pointer prohibit-selection" :class="{ 'speedListAndFather': speed === 0.5 }"
                    @click="pushSpeed(0.5)">0.5</div>
                <div class="speedList mouse-pointer prohibit-selection" :class="{ 'speedListAndFather': speed === 0.7 }"
                    @click="pushSpeed(0.7)">0.7</div>
                <div class="speedList mouse-pointer prohibit-selection" :class="{ 'speedListAndFather': speed === 1.0 }"
                    @click="pushSpeed(1.0)">1.0</div>
                <div class="speedList mouse-pointer prohibit-selection" :class="{ 'speedListAndFather': speed === 2.0 }"
                    @click="pushSpeed(2.0)">2.0</div>
            </div>
            <div class="container showSpeedBox">
                <div v-show="!openSpeedInput" @click="openInput" class="testShowSpeed mouse-pointer prohibit-selection">{{
                    speed }}</div>
                <div v-if="openSpeedInput" class="container">
                    <div class="textInInput  mouse-pointer prohibit-selection" @click="removeInput">取消</div>
                    <input @keyup.enter="testAndPushSpeed(speedInput)" ref="speedInputDom" v-model.trim="speedInput"
                        class="speedInput" placeholder="0.1到3.0">
                    <div v-show="!inputErr" class="font-err-input textInInput mouse-pointer prohibit-selection">确定</div>
                    <div v-show="inputErr" class="textInInput mouse-pointer prohibit-selection"
                        @click="pushSpeed(speedInput)">确定</div>
                </div>

            </div>

        </div>


    </div>
</template>

<style>
.container {
    display: flex;
    align-items: flex-end;
    /* 垂直居底对齐 */
    justify-content: center;
    /* 水平方向居中 */
    position: relative;
}

.showSpeedBox {
    display: flex;
    flex-direction: column;
    /* 将容器的主轴方向设置为垂直方向 */
    justify-content: center;
    /* 水平方向居中 */
    align-items: center;
    /* 垂直方向居中 */
    width: 150px;
    height: 35px;
    margin-bottom: 0px;
    color: #000000;
}

.speedInput {
    margin: 5.5px 5px 3.5px;
    margin-top: 5.5px;
    margin-bottom: 3.5px;
    height: 20px;
    width: 55px;
    border: 1px solid #777777;
    border-radius: 2px;
}

.speedListBox {
    position: absolute;
    display: flex;
    align-items: center;
    /* 垂直居底对齐 */
    justify-content: center;
    /* 水平方向居中 */
    top: 26px;
    height: 45px;
    width: 185px;
}

.speedList {
    padding: 0 8px 0;
    font-size: 15px;
    color: #777777;
}

.speedListAndFather {
    margin-bottom: 2px;
    font-size: 20px;
    color: #000000;
}

.speedCardShowBox {
    display: flex;
    flex-direction: column;
    /* 将容器的主轴方向设置为垂直方向 */
    justify-content: center;
    /* 水平方向居中 */
    align-items: center;
    /* 垂直方向居中 */

}

.testShowSpeed {
    transition: 0.3s ease;
    font-size: 20px;
}

.textInInput {
    transition: 0.3s ease;
    font-size: 18px;
}

.font-err-input {
    text-decoration: line-through;
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
</style>