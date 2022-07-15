<template lang="pug">
.chat 
    .chat__header
      .chat__title {{ title }}
      .close(@click="close") X
    .message
      .message__body

        .message__text(v-for="(item, index) in comments" )
          .photo(
            :width="'50px'"
            :height="'50px'"
          )
          .message__text-body
            .message__header
              p.message__header-author
                span.accent {{item.user.firstName}} {{item.user.lastName}}
                span.time commented at {{ getCommentTime(item) }}
              .open(@click="openEditModal(index)")

                .editModal(v-if="currentActive === index" ref="modal" )
                  ul
                    li(@click="removeComment(index)" v-if="`${adminAccess}` === 'Administrators' || `${getUser.firstName} ${getUser.lastName}` === `${item.user.firstName} ${item.user.lastName}`") Delete
                    li(@click="editComment(index, item.comment)" v-if="`${adminAccess}` === 'Administrators' || `${getUser.firstName} ${getUser.lastName}` === `${item.user.firstName} ${item.user.lastName}`") Edit
                    li(@click="replyOnComment(index, item.comment)") Reply

            .message__text-text(v-html="item.comment")

      .message__input
        //- ckeditor(v-model="editorData" :config="editorConfig" ref="editor")
        //- button.message__button(@clicked="sendMessage" v-if="!!editorData || currentEditableNote !== null" :value="'Send'"  :color="'#4ba5a5'")
        button Send
</template>

<script>
export default {
  name: 'ChatBody',
  props: {
    entityId: {
      type: String,
      default: ''
    },
    title: {
      type: String,
      default: ''
    },
    entityType: {
      type: String,
      default: ''
    },
    conversationId: {
      type: String,
      default: ''
    },
  },
  data() {
    return {
      currentEditableNote: null,
      editorData: "",
      editorConfig: {
        allowedContent: {
          $1: {
            attributes: true,
            styles: true,
            classes: true
          }
        },
        extraPlugins: [ 'colorbutton', 'smiley' ],
        toolbarGroups: [
          { name: 'basicstyles', groups: [ 'basicstyles', 'cleanup' ] },
          { name: 'document', groups: [ 'mode', 'document', 'doctools' ] },
          { name: 'editing', groups: [ 'find', 'selection', 'spellchecker', 'editing' ] },
          { name: 'forms', groups: [ 'forms' ] },
          { name: 'paragraph', groups: [ 'list', 'indent', 'blocks', 'align', 'bidi', 'paragraph' ] },
          { name: 'clipboard', groups: [ 'clipboard', 'undo' ] },
          { name: 'links', groups: [ 'links' ] },
          { name: 'insert', groups: [ 'insert' ] },
          { name: 'styles', groups: [ 'styles' ] },
          { name: 'colors', groups: [ 'colors' ] },
          { name: 'tools', groups: [ 'tools' ] },
          { name: 'others', groups: [ 'others' ] },
          { name: 'about', groups: [ 'about' ] }
        ],
        removeButtons: 'Source,Save,NewPage,ExportPdf,Preview,Print,Templates,Cut,Copy,Paste,PasteText,PasteFromWord,Find,Replace,SelectAll,Form,Checkbox,Radio,TextField,Textarea,Select,ImageButton,HiddenField,Button,Superscript,Subscript,CopyFormatting,NumberedList,Blockquote,CreateDiv,JustifyLeft,JustifyCenter,JustifyRight,JustifyBlock,BidiLtr,BidiRtl,Language,Anchor,HorizontalRule,Table,Flash,PageBreak,Iframe,Styles,Format,Font,FontSize,ShowBlocks,Maximize,About',
        uiColor: "#ffffff",
        height: 200
      },

      isEditModal: false,
      currentActive: -1,
      editableItem: -1,
      clicks: 0,
      // admin: '',

      currentConversationId: this.conversationId,
      comments: [],
      getUser: {}
    }
  },
  computed: {
    adminAccess() {
      return this.getUser.group.name
    },

  },
  methods: {
    finishExecutor(){
      this.currentActive = -1
    },
    ClickOutside(e) {
      console.log(this.$refs)
      console.log(e.target)
      if (!this.$refs.modal[0].contains(e.target) && e.target.className !== 'message__header-button') {
        this.currentActive = -1
      }
    },
    getCommentTime({createdAt}){
      console.log(createdAt)
      return new Date()
    //   return moment(createdAt).format('DD/MM/YYYY, HH:mm')
    },
    close() {
      this.$emit('close')
    },
    removeComment(index) {
      this.comments.splice(index, 1)
      this.clicks = 0
      this.saveComments()
      this.finishExecutor()
    },
    editComment(index, comment) {
      this.finishExecutor()
      console.log(this.currentActive)
      this.currentActive = -1
      this.editableItem = index
      this.clicks = 0
      this.editorData = comment
    },
    replyOnComment( comment) {
      this.editorData = `<blockquote style="overflow:hidden;height:70px;border-left: 5px solid #8fa3ad;padding:0 0 0 20px; opacity:0.5;" contenteditable="false">${comment.substring(0, 300)}...</blockquote> </br>`
      // this.editorData = `<blockquote style="overflow:hidden;height:70px;" contenteditable="false">${comment.substring(0, 300)}...</blockquote> </br>`
    },
    sendMessage() {
      if(this.editableItem === -1) {
        this.comments.push({comment: this.editorData, user: this.getUser})
        this.editorData = ''
        this.editableItem = -1
        this.$nextTick(this.scrollToBottom);
        this.saveComments()
        console.log(this.getUser)

        return
      }
      const i = this.editableItem
      this.comments[i].comment = this.editorData
      console.log(this.getUser)
      this.editorData = ''
      this.editableItem = -1
      this.$nextTick(this.scrollToBottom);
      this.saveComments()
    },
    openEditModal(index) {
      if(this.currentActive > 0) this.currentActive = -1
      this.currentActive = index
      // this.clicks += 1
      // if (this.clicks > 1) {
      //   this.currentActive = -1
      //   this.clicks = 0
      // }
      document.addEventListener('click', this.ClickOutside)
    },
    scrollToBottom() {
      const container = this.$el.querySelector(".message__body");
      container.scrollTop = container.scrollHeight;
    },
    async getComments() {
      try {
        if(!this.currentConversationId) return
       const res = await this.$http.get(`/api/v1/conversation/${this.currentConversationId}`)
        this.initData(res)
      } catch(e) {
        this.alertToggle({ message: "Error on getting comments", isShow: true, type: "error" })
      }
    },
    async saveComments() {
      try {
        if(!this.currentConversationId) this.currentConversationId = null
        const res = await this.$http.put(`/api/v1/conversation/${this.currentConversationId}`, {
          comments:  this.comments,
          _entityId: this.entityId,
          entityType: this.entityType,
        })
        this.initData(res)
        console.log(33)
        // this.alertToggle({ message: "Comment successfully saved", isShow: true, type: "success" })
      } catch(e) {
        // this.alertToggle({ message: "Comment didn't saved", isShow: true, type: "error" })
      }
    },
    initData({data}) {
      this.currentConversationId = data._id
      this.comments = data.comments
    }
  },
  async created() {
    await this.getComments()
    this.$nextTick(this.scrollToBottom);
  },
  beforeUnmount() {
    document.removeEventListener('click', this.ClickOutside)
  }
  // mounted() {
  //   this.$nextTick(this.scrollToBottom);
  //
  // }
}
</script>

<style lang="scss" scoped>

.chat {
//   position: fixed;
//   background-color: lightcoral;
//   z-index: 43;
//   top: 0;
//   right: 0;
//   left: 255px;
//   bottom: 0;
//   height: 100%;
    position: fixed;
    background-color: lightcoral;
    z-index: 43;
    top: 0;
    height: 100%;
    width: 100%;

  &__header {
    height: 60px;
    position: fixed;
    border-bottom: 1px solid lightblue;
    width: 87%;
    padding: 20px;
    padding-left: 105px;
    z-index: 43;
    background-color: lightblue;
    box-sizing: border-box;
  }
  &__title {
    font-size: 16px;
    font-family: Myriad600;
    color: #333;
  }
}
.close {
  transform: translateY(-50%);
  top: 50%;
  right: 50px;
}
.message {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding-top: 60px;

}
.message__body {
  flex-grow: 1;
  padding: 50px 50px 30px 30px;
  height: 70%;
  overflow-y: auto;
}
.message__text {
  display: flex;
  gap: 20px;
  position: relative;

  &::before {
    //position: absolute;
    //top: 11px;
    //right: 100%;
    //left: -8px;
    //width: 8px;
    //height: 16px;
    //pointer-events: none;
    //content: " ";
    //clip-path: polygon(0 50%, 100% 0, 100% 100%);
    position: absolute;
    top: 10px;
    left: 60px;
    width: 10px;
    height: 20px;
    pointer-events: none;
    content: " ";
    clip-path: polygon(0 50%, 100% 0, 100% 100%);
    //Z-INDEX: 2222;
    //border: 1px solid $border;
    background-color: lightblue;
  }

  &:not(:last-child) {
    margin-bottom: 30px;
    position: relative;

    &::after {
      content: " ";
      position: absolute;
      top: 100%;
      left: 100px;
      height: 30px;
      width: 2px;
      background-color: lightblue;
    }
  }

  &-body {
    border: 1px solid gray;
    position: relative;
    border-radius: 6px;
    background-color: white;
    flex-grow: 1;
    width: 100%;
    overflow-x: auto;
    //min-height: 91px;
  }

  &-text {
    //padding-right: 16px;
    //padding-left: 16px;
    padding: 18px;
    color: #333;
  }

}
.message__header {
  display: flex;
  align-items: center;
  //padding-right: 16px;
  //padding-left: 16px;
  justify-content: space-between;
  background-color: lightblue;
  border-bottom: 1px solid gray;
  //border-top-left-radius: 6px;
  //border-top-right-radius: 6px;
  padding: 3px 16px;
  //padding-top: 3px;
  //padding-bottom: 3px;

  &-author {
    margin: 0;
    padding-top: 8px;
    padding-bottom: 8px;

    .accent {
      font-size: 16px;
      font-family: Myriad600;
      margin-right: 8px;
      color: #333;
    }

    .time {
      color: #od1616;
    }
  }
  &-button {
    font-size: 30px;
    padding-bottom: 15px;
    cursor: pointer;
    //padding-right: 120px;
    //position: relative;
  }
}
.message__input {
  position: relative;
}

.message__button {
  position: absolute;
  right: 20px;
  top: 6px;

}

.editModal {
  border: 1px solid gray;
  box-sizing: border-box;
  position: absolute;
  top: -5px;
  right: 40px;
  width: 100px;
  height: 90px;
  //text-align: center;
  padding: 7px;
  background-color: white;
  color: #333;

  ul {
    list-style: none;
    padding: 0;
    margin: 0;
    font-size: 14px;

    li {
      padding: 5px;
      cursor: pointer;
      &:hover {
        color: green;
      }
    }
  }
}
</style>
