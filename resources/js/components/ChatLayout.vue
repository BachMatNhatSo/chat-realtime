<template>
    <div>
        <div class="chat">
            <div class="chat-title">
                <h1>Chatroom</h1>
            </div>
            <div class="messages" ref="messageContainer">
                <div class="messages-content">
                    <ChatItem v-for="(message, index) in list_messages" :key="index" :message="message"></ChatItem>
                </div>
            </div>
            <div class="message-box">
                <input type="text" v-model="message" @keyup.enter="sendMessage" class="message-input"
                    placeholder="Type message..." />
                <button type="button" class="message-submit" @click="sendMessage">Send</button>
            </div>
        </div>
        <div class="bg"></div>
    </div>

</template>
<script setup>
import { ref, onBeforeMount, nextTick } from 'vue';
import ChatItem from './ChatItem.vue'

const message = ref('')
const list_messages = ref([])
const messageContainer = ref(null)
onBeforeMount(() => {
    loadMessage();
    Echo.channel('laravel_database_chatroom')
        .listen('MessagePosted', (data) => {
            const message = data.message
            message.user = data.user
            list_messages.value.push(message)
            scrollToBottom();
        })

});

async function loadMessage() {
    try {
        const response = await axios.get('/messages')
        list_messages.value = response.data
        await nextTick();
        scrollToBottom();
    } catch (error) {
        console.log(error)
    }
}

async function sendMessage() {
    try {
        const response = await axios.post("/messages", {
            message: message.value,
        });
        list_messages.value.push(response.data.message);
        message.value = "";
        nextTick(() => {
            scrollToBottom();
        });
    } catch (error) {
        console.log(error);
    }
}

function scrollToBottom() {
    // Sử dụng nextTick để đảm bảo DOM đã được cập nhật trước khi cuộn
    nextTick(() => {
        if (messageContainer.value) {
            messageContainer.value.scrollTop = messageContainer.value.scrollHeight;
        }
    });
}

</script>

<style lang="scss" scoped>
.messages {
    height: 80%;
    overflow-y: scroll;
    padding: 0 20px;
}

/*--------------------
Body
--------------------*/
.bg {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    z-index: 1;
    background: url('https://images.unsplash.com/photo-1451186859696-371d9477be93?crop=entropy&fit=crop&fm=jpg&h=975&ixjsv=2.1.0&ixlib=rb-0.3.5&q=80&w=1925') no-repeat 0 0;
    filter: blur(80px);
    transform: scale(1.2);
}

/*--------------------
Chat
--------------------*/
.chat {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 500px;
    height: 80vh;
    max-height: 700px;
    z-index: 2;
    overflow: hidden;
    box-shadow: 0 5px 30px rgba(0, 0, 0, .2);
    background: rgba(0, 0, 0, .5);
    border-radius: 20px;
    display: flex;
    justify-content: space-between;
    flex-direction: column;
}

/*--------------------
Chat Title
--------------------*/
.chat-title {
    flex: 0 1 45px;
    position: relative;
    z-index: 2;
    background: rgba(0, 0, 0, 0.2);
    color: #fff;
    text-transform: uppercase;
    text-align: left;
    padding: 10px 10px 10px 50px;

    h1,
    h2 {
        font-weight: normal;
        font-size: 16px;
        margin: 0;
        padding: 0;
    }

    h2 {
        color: rgba(255, 255, 255, .5);
        font-size: 8px;
        letter-spacing: 1px;
    }

    .avatar {
        position: absolute;
        z-index: 1;
        top: 8px;
        left: 9px;
        border-radius: 30px;
        width: 30px;
        height: 30px;
        overflow: hidden;
        margin: 0;
        padding: 0;
        border: 2px solid rgba(255, 255, 255, 0.24);

        img {
            width: 100%;
            height: auto;
        }
    }
}

/*--------------------
Message Box
--------------------*/
.message-box {
    flex: 0 1 40px;
    width: 100%;
    background: rgba(0, 0, 0, 0.3);
    padding: 10px;
    position: relative;

    & .message-input {
        background: none;
        border: none;
        outline: none !important;
        resize: none;
        color: rgba(255, 255, 255, .7);
        font-size: 11px;
        height: 17px;
        margin: 0;
        padding-right: 20px;
        width: 265px;
    }

    textarea:focus:-webkit-placeholder {
        color: transparent;
    }

    & .message-submit {
        position: absolute;
        z-index: 1;
        top: 9px;
        right: 10px;
        color: #fff;
        border: none;
        background: #248A52;
        font-size: 10px;
        text-transform: uppercase;
        line-height: 1;
        padding: 6px 10px;
        border-radius: 10px;
        outline: none !important;
        transition: background .2s ease;

        &:hover {
            background: #1D7745;
        }
    }
}
</style>