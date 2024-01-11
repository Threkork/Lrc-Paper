<script setup>
import { ref, onMounted, watch } from 'vue';

const smooth = ref(true)

const container = ref(null);
const draggable = ref(null);
const isDragging = ref(false);
const position = ref(0);

const beginNum = ref(0)
const beginIsPlay = ref(false)

const emit = defineEmits(['progresssonPosition', 'progresssonplay', 'progresssonpause'])

const pushPosition = () => {
    emit('progresssonPosition', position.value)
}
const play = () => {
    emit('progresssonplay')
}
const pause = () => {
    emit('progresssonpause')
}

const startDragging = (e) => {
    isDragging.value = true;

    const containerRect = container.value.getBoundingClientRect();


    const handleMouseDown = (e) => {

        if (isDragging.value) {
            beginNum.value = beginNum.value + 1
            if (beginNum.value === 2) {
                pause()
            }

            const x = e.clientX - containerRect.left;
            const percentage = (x / containerRect.width) * 100;

            // 限制拖动范围在 [0, 100] 之间
            position.value = Math.max(0, Math.min(100, percentage));
            pushPosition()
        }
    };

    const handleMouseMove = (e) => {
        smooth.value = false
        if (isDragging.value) {
            beginNum.value = beginNum.value + 1
            if (beginNum.value === 2) {
                pause()
            }
            const x = e.clientX - containerRect.left;
            const percentage = (x / containerRect.width) * 100;

            // 限制拖动范围在 [0, 100] 之间
            position.value = Math.max(0, Math.min(100, percentage));
            pushPosition()

        }
    };

    const handleMouseUp = () => {
        isDragging.value = false;
        smooth.value = true
        document.removeEventListener('mousemove', handleMouseMove);
        document.removeEventListener('mouseup', handleMouseUp);
        if (beginIsPlay.value) {
            play()
        }
        beginNum.value = 0
    };

    document.addEventListener('mousedown', handleMouseDown)
    document.addEventListener('mousemove', handleMouseMove);
    document.addEventListener('mouseup', handleMouseUp);
};


onMounted(() => {
    // 初始化拖动元素的位置
    const containerRect = container.value.getBoundingClientRect();
    const draggableRect = draggable.value.getBoundingClientRect();
    position.value = ((draggableRect.left - containerRect.left) / containerRect.width) * 100;
});


const props = defineProps({
    changePosition: Number,
    audioFormatCurrent: Number,
    audioFormatDuration: Number,
    fileName: String,
    isPlay: Boolean,
})


watch(
    () => props.changePosition,

    (newVal) => {
        position.value = newVal
    }
)

watch(
    () => beginNum.value,
    (newVal, oldVal) => {
        if (oldVal === 0) {
            beginIsPlay.value = props.isPlay
            console.log(beginIsPlay.value);
        }
    }
)




</script>


<template>
    <div>
        <div class="time">
            <p class="left-element" :class="{ 'prohibit-selection': isDragging }">{{ audioFormatCurrent }}</p>
            <div class="title" :class="{ 'prohibit-selection': isDragging }">
                <h3>{{ fileName }}</h3>
            </div>
            <p class="right-element" :class="{ 'prohibit-selection': isDragging }">{{ audioFormatDuration }}</p>
        </div>
        <div ref="container" class="container" @mousedown="startDragging">
            <div class="background"></div>

            <div ref="draggable" class="draggable" :style="{ width: `${position}%` }" :class="{ 'smooth': smooth }">
            </div>

            <div class="progress-thumb" :style="{ left: `${position}%` }" :class="{ 'smooth': smooth }"></div>
        </div>
    </div>
</template>



<style scoped>
.time {
    display: flex;
    justify-content: space-between;
    align-items: center;

}

.left-element {
    width: 65px;
}



.container {
    display: flex;
    align-items: center;

    /* padding-top: 7px; */
    position: relative;
    width: 70vw;
    height: 20px;
    overflow: visible;
    cursor: pointer;
}

.background {
    position: absolute;
    width: 70vw;
    height: 3px;
    background-color: #09090944;
    /* cursor: pointer; */

}

.draggable {
    left: 0;
    position: absolute;
    height: 3px;
    background-color: #484848;
    cursor: pointer;

}

.progress-thumb {
    position: absolute;
    width: 14px;
    height: 14px;
    border-radius: 50%;
    background: #FFFFFF;
    box-shadow: 0 0 1px 1px #000000;
}

.prohibit-selection {
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

.smooth {
    transition: 0.3s ease;
}
</style>
