<script setup>
const { isMobile } = useDevice()

const { $i18n } = useNuxtApp()

const colorMode = useColorMode()

const props = defineProps({
    sendMessage: {
        type: Function,
        required: true
    },
    disabled: {
        type: Boolean,
        default: false
    },
    loading: {
        type: Boolean,
        default: false
    },
    scrollDown: {
        type: Function,
        required: true
    },
    needScroll: {
        type: Function,
        required: true
    }
})

const message = ref('')
const rows = ref(1)
const autoGrow = ref(true)
const textArea = ref()
const maxHeight = ref(0)
const hint = computed(() => {
    return isMobile ? '' : $i18n.t('pressEnterToSendYourMessageOrShiftEnterToAddANewLine')
})
const watchflag = ref(true)
const shouldShow = ref(false)

onMounted(async () => {
    // 获取文本框的 DOM 元素
    const textAreaElement = textArea.value.$el.getElementsByTagName('textarea')[0]
    // 解决删除的时候行高变化慢一拍的问题，模拟一个信号
    function adjustTextAreaHeight() {
        textAreaElement.style.height = 'auto'
        textAreaElement.style.height = `${textAreaElement.scrollHeight}px`
    }
    maxHeight.value = parseInt(window.getComputedStyle(textAreaElement).maxHeight)
    textAreaElement.addEventListener('keydown', function (event) {
        if (event.key === 'Backspace' || event.key === 'Delete' || event.key === 'Enter') {
            setTimeout(adjustTextAreaHeight, 0)
        }
    })
    textAreaElement.addEventListener('click', function (event) {
        setTimeout(adjustTextAreaHeight, 0)
    })
    await nextTick()
    window.addEventListener('click', function (event) {
        setTimeout(adjustTextAreaHeight, 0)
    })
    window.addEventListener('resize', function (event) {
        setTimeout(adjustTextAreaHeight, 0)
    })
    window.addEventListener('scroll', () => {
        shouldShow.value = props.needScroll()
    });
})

watch(message, () => {
    // 获取文本框元素
    if (watchflag.value) {
        const textAreaElement = textArea.value.$el.getElementsByTagName('textarea')[0]
        if (message.value === '') {
            textAreaElement.style.height = 'auto'
            textAreaElement.style.height = `0px`
        } else {
            // 重置文本框的高度
            textAreaElement.style.height = 'auto'
            // 设置文本框的高度为其滚动内容的高度
            textAreaElement.style.height = `${textAreaElement.scrollHeight}px`
        }
        // 如果文本框的内容高度大于maxHeight.value，那么显示滚动条；如果小于maxHeight.value，那么隐藏滚动条
        textAreaElement.style.overflowY =
            textAreaElement.scrollHeight > maxHeight.value ? 'auto' : 'hidden'
        autoGrow.value = textAreaElement.scrollHeight <= maxHeight.value
    } else {
        watchflag.value = true
    }
})
const send = () => {
    let msg = message.value
    usePrompt('')
    // remove the last "\n"
    if (msg[msg.length - 1] === '\n') {
        msg = msg.slice(0, -1)
    }
    if (msg.length > 0) {
        props.sendMessage(msg)
    }
}

const focus = () => {
    nextTick(() => {
        textArea.value.focus()
    })
}

// usePrompt是一个赋值函数，能够给输入框正确赋值，用它！
const usePrompt = async (prompt) => {
    watchflag.value = false
    message.value = prompt
    const textAreaElement = textArea.value.$el.getElementsByTagName('textarea')[0]
    autoGrow.value = false
    await nextTick()
    textAreaElement.style.height = 'auto'
    textAreaElement.style.height = `${Math.min(textAreaElement.scrollHeight, maxHeight.value)}px`
    autoGrow.value = textAreaElement.scrollHeight <= maxHeight.value
    textAreaElement.style.overflowY =
        textAreaElement.scrollHeight > maxHeight.value ? 'auto' : 'hidden'
    textArea.value.focus()
}

const clickSendBtn = () => {
    send()
}

const clickScrollBtn = () => {
    props.scrollDown()
}

const enterOnly = (event) => {
    event.preventDefault()
    if (!isMobile) {
        if (event.isComposing) {
            // 当前正在进行输入法组合输入
            // 什么也不做
        } else {
            send()
            // 当前没有正在进行输入法组合输入
        }
    } else {
        // 手机上回车只做换行操作
        const textarea = textArea.value
        const start = textarea.selectionStart
        const end = textarea.selectionEnd
        const value = textarea.value
        message.value = `${value.substring(0, start)}\n${value.substring(end)}`
        // 使用 nextTick, 等待 watch message 执行了再定位光标
        nextTick(() => {
            textarea.setSelectionRange(start + 1, start + 1)
        })
    }
}

defineExpose({
    usePrompt,
    focus
})
</script>

<template>
    <div class="flex-grow-1 d-flex align-center justify-space-between">
        <div class="input-wrapper">
            <v-textarea
                ref="textArea"
                v-model="message"
                :placeholder="hint"
                :rows="rows"
                max-rows="8"
                :auto-grow="autoGrow"
                :disabled="disabled"
                :loading="loading"
                :hide-details="true"
                class="input-textarea"
                variant="outlined"
                :density="isMobile ? 'compact' : 'default'"
                append-inner-icon="fa:fa-solid fa-paper-plane"
                @keydown.enter.exact="enterOnly"
                @click:append-inner="clickSendBtn"
            ></v-textarea>
            <v-icon v-show="shouldShow" 
                icon="arrow_circle_down" 
                class="scroll-down-icon" 
                @click="clickScrollBtn"
                :color="$colorMode.value === 'light' ? 'black' : 'white'"
                >
            </v-icon>
            <div class="text-under-textarea">
                <span
                    >Free Research Preview. Powered by OpenAI. Sponsored by the
                    <a href="https://zjuvag.org/" target="_blank" rel="noreferrer" class="underline"
                        >VAI</a
                    >
                    Group, State Key Lab of CAD&CG, Zhejiang University.</span
                >
            </div>
        </div>
    </div>
</template>

<style scoped>
.input-wrapper {
    display: flex;
    align-items: stretch;
    flex-direction: column;
    width: 100%;
}

.scroll-down-icon {
    position: absolute;
    right: 60px;
    bottom: 55px;
    color: #38235b;
}

.text-under-textarea {
    text-align: center;
    padding: 0 1rem 0 1rem; /* top, right, bottom, left */
    margin-top: 4px;
    font-size: 10px;
    margin-bottom: 0px;
}
</style>
