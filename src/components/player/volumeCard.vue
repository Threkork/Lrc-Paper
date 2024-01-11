<script setup>
import { ref, onMounted, watch, computed } from 'vue'

const smooth = ref(true)
const container = ref(null)
const draggable = ref(null)
const isDragging = ref(false)
const position = ref(0)



const emit = defineEmits(['volumesonPosition'])

const pushPosition = () => {
    emit('volumesonPosition', parseFloat((position.value / 100).toFixed(2)))
}

const showVolume = computed(() => {
    let i = 0


    i = Math.round(props.changeVolume * 100)
    return `${i}%`



})

const startDragging = (e) => {
    isDragging.value = true

    const containerRect = container.value.getBoundingClientRect()

    const handleMouseDown = (e) => {
        if (isDragging.value) {
            const y = containerRect.bottom - e.clientY;
            const percentage = (y / containerRect.height) * 100;

            // 限制拖动范围在 [0, 100] 之间
            position.value = Math.max(0, Math.min(100, percentage))
            pushPosition()
        }
    }

    const handleMouseMove = (e) => {
        smooth.value = false
        if (isDragging.value) {
            const y = containerRect.bottom - e.clientY;
            const percentage = (y / containerRect.height) * 100;

            // 限制拖动范围在 [0, 100] 之间
            position.value = Math.max(0, Math.min(100, percentage))
            pushPosition()
        }
    }

    const handleMouseUp = () => {
        isDragging.value = false
        smooth.value = true
        document.removeEventListener('mousemove', handleMouseMove)
        document.removeEventListener('mouseup', handleMouseUp)
    }

    document.addEventListener('mousedown', handleMouseDown)
    document.addEventListener('mousemove', handleMouseMove)
    document.addEventListener('mouseup', handleMouseUp)
}

onMounted(() => {
    // 初始化拖动元素的位置
    const containerRect = container.value.getBoundingClientRect();
    const draggableRect = draggable.value.getBoundingClientRect();
    position.value = ((containerRect.bottom - draggableRect.bottom) / containerRect.height) * 100;
})

const props = defineProps({
    changeVolume: Number,
})

watch(
    () => props.changeVolume,
    (newVal) => {
        position.value = newVal * 100
    },
    { immediate: true }
)

</script>

<template>
    <div class="volumebox">
        <div class="show prohibit-selection">{{ showVolume }}</div>
        <div ref="container" class="container" @mousedown="startDragging">
            <div class="background"></div>
            <div ref="draggable" class="draggable" :style="{ height: `${position}%` }" :class="{ 'smooth': smooth }"></div>
            <div class="progress-thumb" :style="{ top: `${-position + 92.5}%` }" :class="{ 'smooth': smooth }"></div>

        </div>
    </div>
</template>

<style scoped>
.volumebox {
    display: flex;
    flex-direction: column;
    /* 将容器的主轴方向设置为垂直方向 */
    justify-content: center;
    /* 水平方向居中 */
    align-items: center;
    /* 垂直方向居中 */
    width: 50px;
}

.show {
    display: flex;
    padding-bottom: 10px;
    font-size: 18px;
    margin: 0;
}

.container {
    display: flex;
    align-items: flex-end;
    /* 垂直居底对齐 */
    justify-content: center;
    /* 水平方向居中 */
    position: relative;
    width: 37px;
    /* 修改宽度为垂直条的宽度 */
    height: 110px;
    /* 修改高度为垂直条的高度 */
    overflow: visible;
    cursor: pointer;

}

.background {
    position: absolute;
    width: 3px;
    /* 修改为垂直条的宽度 */
    height: 110px;
    /* 修改为垂直条的高度 */
    background-color: #09090944;
}

.draggable {
    position: absolute;
    width: 3px;
    /* 修改为垂直条的宽度 */
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
    -webkit-user-select: none;
    -ms-user-select: none;
    -khtml-user-select: none;
    user-select: none;
}

.smooth {
    transition: 0.15s ease;
}
</style>
