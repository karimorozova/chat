<template lang="pug">
.chat-wrapper
    button(@click="openChart") Open Chat
    .div Open chart
    


    ChatBody(
      v-if="isChat"
      @close="closeChat"
      :entityId="entityId"
      :title="title"
      :entityType="entityType"
      :conversationId="conversationId"
    )
</template>

<script>
import ChatBody from './ChatBody.vue'
// import ChatBody from "./ChatBody";
export default {
    name: "ChatConversation",
    //   components: { ChatBody,},
    props: {
        entityId: {
            type: String,
            default: "",
        },
        title: {
            type: String,
            default: "",
        },
        entityType: {
            type: String,
            default: "",
        },
        conversationId: {
            type: String,
            default: "",
        },
    },
    data() {
        return {
            isChat: false,
        };
    },
    methods: {
        openChart() {
            this.isChat = true;
            const el = document.querySelector("body");
            el.style.overflow = "hidden";
        },
        closeChat() {
            this.isChat = false;
            const el = document.querySelector("body");
            el.style.overflow = "auto";
        }
    },
    components: { ChatBody }
}
</script>

<style lang="scss" scoped>

</style>
