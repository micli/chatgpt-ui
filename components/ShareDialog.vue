<script setup>
import ChatGPTLogo from '@/assets/gpt.svg'
import html2canvas from 'html2canvas'
const currentModel = useCurrentModel()
const props = defineProps({
    appBar: {
        type: Boolean,
        required: true
    },
    conversation: {
        type: Object,
        required: true
    }
})

const { isMobile } = useDevice()
const user = useUser()
const dialog = ref(false)
const hideUserName = ref(true)
onNuxtReady(() => {
    currentModel.value = getCurrentModel()
})
const bgColor = computed(() => {
    // return currentModel.value.name === 'gpt-4' ? 'rgb(27,27,27)' : 'rgb(25, 195, 125)';
    return currentModel.value.name === 'gpt-4' ? 'rgb(56, 35, 91)' : 'rgb(170,144,212)'
})
const getDate = () => {
    const now = new Date()
    return now.toLocaleDateString('en-US')
}

const shareAsImage = () => {
    const element = document.getElementById('divToShare')
    // 复制一份, 用于修改样式
    var copyDOM = element.cloneNode(true)
    // 删除 box-shadow, html2canvas 不支持 box-shadow
    copyDOM.classList.remove('share-window-shadow')
    // 删除 share-panel, 使得可以截取全部内容, 避免受滚动条影响
    var divToScroll = copyDOM.querySelector('#divToScroll')
    divToScroll.classList.remove('share-panel')
    // 删除匿名按钮
    var btnElement = copyDOM.querySelector('#hideUserNameID')
    if (btnElement) {
        btnElement.parentNode.removeChild(btnElement)
        var infoElement = copyDOM.querySelector('.share-info')
        infoElement.style.setProperty('margin-bottom', '10px')
    }
    // 把 copyDOM 添加到 document 中
    document.querySelector('body').appendChild(copyDOM)

    html2canvas(copyDOM, {
        allowTaint: true,
        useCORS: true,
        width: element.clientWidth - 50,
        windowWidth: element.clientWidth
    }).then((canvas) => {
        const imageDataURL = canvas.toDataURL('image/png')
        const link = document.createElement('a')
        link.href = imageDataURL
        link.download = 'share_conversation.png' // 指定下载文件名
        // 模拟点击链接进行下载
        link.click()
        // 从 document 中删除 copyDOM
        copyDOM.remove()
    })
}
</script>

<template>
    <v-dialog v-if="appBar" v-model="dialog" persistent class="dialog-width">
        <template v-slot:activator="{ props }">
            <v-btn
                v-bind="props"
                :title="$t('share')"
                icon="share"
                variant="text"
                class="mr-4"
            ></v-btn>
        </template>
        <v-card>
            <v-toolbar>
                <v-toolbar-title>{{ $t('shareConversation') }}</v-toolbar-title>
                <v-spacer></v-spacer>
                <v-btn icon="close" variant="text" @click="dialog = false"></v-btn>
            </v-toolbar>
            <div class="d-flex justify-between">
                <v-card-text v-show="!conversation.shared" class="flex-grow-1 text-left">{{
                    $t('shareTips3')
                }}</v-card-text>

                <v-card-actions class="align-self-center">
                    <v-btn
                        prepend-icon="fa:fa-solid fa-download"
                        variant="outlined"
                        @click="shareAsImage"
                        >{{ $t('share') }}</v-btn
                    >
                </v-card-actions>
            </div>

            <!-- <v-card-text v-show="conversation.shared">{{ $t("shareTips2") }}</v-card-text> -->
            <div id="divToShare" class="share-window share-window-shadow">
                <div class="share-option">
                    <div class="share-topic">{{ conversation.topic }}</div>
                    <div class="share-info">
                        <span v-if="!hideUserName" class="mr-2">{{ user.username }} · </span>
                        <span>{{ getDate() }}</span>
                        <v-spacer></v-spacer>
                        <div id="hideUserNameID" class="d-flex align-center">
                            <v-switch
                                v-model="hideUserName"
                                inline
                                color="primary"
                                hide-details=""
                            ></v-switch>
                            <span class="ml-3">{{ $t('showUserName') }}</span>
                        </div>
                    </div>
                </div>
                <div
                    id="divToScroll"
                    class="d-flex flex-column justify-space-between share-panel"
                    style="width: 100%"
                >
                    <div class="d-flex flex-column flex-grow-1 ml-4 mr-4">
                        <div
                            v-for="(message, index) in conversation.messages"
                            :key="index"
                            class="d-flex align-start"
                            :class="message.is_bot ? 'justify-start' : 'justify-end'"
                        >
                            <div
                                v-if="message.is_bot && !isMobile"
                                class="avatar-bot"
                                :style="{ backgroundColor: bgColor }"
                            >
                                <img :src="ChatGPTLogo" alt="ChatGPT" />
                            </div>
                            <MsgContent
                                :message="message"
                                :message-index="index"
                                :use-prompt="() => {}"
                                :retry-message="() => {}"
                                :delete-message="() => {}"
                                :style="`max-width: ${isMobile ? '95%' : '75%'};`"
                                :editable="false"
                            />
                            <div v-if="!message.is_bot && !isMobile" class="avatar-user">
                                <div class="avatar-text">
                                    <v-icon icon="fa:fa-solid fa-user" size="28"></v-icon>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </v-card>
    </v-dialog>
</template>

<style scoped>
@media screen and (max-width: 380px) {
    .dialog-width {
        max-width: 350px;
    }
}
@media screen and (min-width: 381px) {
    .dialog-width {
        max-width: 768px;
    }
}
@media screen and (min-width: 769px) {
    .dialog-width {
        max-width: 1024px;
    }
}
@media screen and (min-width: 1025px) {
    .dialog-width {
        max-width: 1280px;
    }
}
.share-window {
    border-radius: 10px;
    border: 1px solid rgba(161, 158, 158, 0.5);
    margin: 10px 25px 20px 25px;
}
.share-window-shadow {
    box-shadow: 0 0 5px rgba(170, 189, 181, 0.305);
}
.share-panel {
    max-height: 400px;
    overflow-y: auto;
    padding: 0 0 15px 0;
}
.share-option {
    font-size: 1.1rem;
    border-bottom: 1px solid rgba(161, 158, 158, 0.5);
    padding: 15px 30px 0 30px;
}
.share-topic {
    font-weight: bold;
}
.share-info {
    color: gray;
    display: flex;
    align-items: center;
    justify-content: space-between;
}
.avatar-bot {
    margin: 22px 10px 0 10px;
    padding: 8px 8px 2px 8px;
    border-radius: 10px;
    background-color: rgb(25, 195, 125);
    transform: scale(0.7);
}
.avatar-user {
    margin: 26px 10px 0 10px;
    border-radius: 10px;
    background-color: rgb(178, 207, 130);
    transform: scale(0.8);
}
.avatar-text {
    width: 51px;
    height: 51px;
    color: white;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 1.5rem;
}
</style>
