<template>
  <transition name="bounce">
    <div v-if="isCollapsedChat"
         @click="openChat">
      <div class="heart">
        <h2 v-if="unreadByUser">
          <span class="unread_by_user">
            +{{ unreadByUser }}
          </span>
        </h2>
        <img v-else src="@/assets/icons/common/heartbeat.svg" alt="">
      </div>
    </div>
    <v-card v-if="!isCollapsedChat"
            id="liveChat"
            class="user_live_chat">
      <v-card-title class="chat_header_user">
        <el-button @click="closeChat" type="text" class="closeChat">
          <v-icon class="white--text">close</v-icon>
        </el-button>
        <h3 class="pl-3 white--text">
          Live Chat
          <span class="pl-3">|</span>
        </h3>
        <transition name="fade">
            <span v-if="isTypingAdmin" class="pl-4 white--text">
                <span>Admin</span>
                ...<v-icon size="medium" class="white--text">edit</v-icon>
            </span>
          <span v-if="!isTypingAdmin && isOnlineAdmin" class="pl-4 white--text">
              Admin online
            </span>
          <span v-if="!isOnlineAdmin" class="pl-4 white--text">
              Admin offline
            </span>
        </transition>
      </v-card-title>
      <v-card-text v-if="chatMessages"
                   ref="chatMessages"
                   class="user_chat_messages">
          <span v-if="Object.keys(chatMessages).length === 0">
            <h2 class="pt-5 info--text">Have question?</h2>
          </span>
        <div v-for="(chat, key) in chatMessages"
             :key="key">
          <el-row>
            <el-col :span="24" class="info--text chat_msg_meta">
                <span :class="chat.creator ? 'left' : 'right'">
                <span>{{ chat.creator ? 'You' : 'ReHigh' }}:</span>
                {{ new Date(chat.date) | chatTime }}
                </span>
            </el-col>
          </el-row>
          <el-row :class="chat.creator ? 'left' : 'right'">
            <el-col :span="24">
              <p :class="chat.creator ? 'pr-4 primary--text' : 'pl-4 success--text'"
                 class="chat_msg">{{ chat.msg }}</p>
            </el-col>
          </el-row>
        </div>
      </v-card-text>
      <v-divider></v-divider>
      <v-card-actions>
      <textarea v-model="msg"
                ref="msgInput"
                cols="100" rows="3"
                placeholder="Type..."
                class="chat_input"
                @input="detectTyping"
                @keydup.shift.enter="msg+='\n'"
                @keydup.ctrl.enter="msg+='\n'"
                @keydup.alt.enter="msg+='\n'"
                @keydup.meta.enter="msg+='\n'"
                @keydup.down="msg+='\n'"
                @keyup.enter.exact="sendChatMessage">
      </textarea>
      </v-card-actions>
    </v-card>
  </transition>
</template>

<script>
export default {
  name: 'UserLiveChat',
  props: ['chatId', 'isCollapsed'],
  data () {
    return {
      msg: '',
      isTyping: false,
      isCollapsedChat: this.isCollapsed
    }
  },
  methods: {
    openChat () {
      this.isCollapsedChat = false
      this.setCollapse()
      this.setUnread('unreadByUser', 0)
      this.$nextTick(function () {
        this.scrollToBottom()
      })
    },
    closeChat () {
      this.isCollapsedChat = true
      this.setCollapse()
    },
    sendChatMessage () {
      if (this.msg.trim()) {
        this.$store.dispatch('sendChatMessage', {
          chatId: this.chatId,
          msg: this.msg,
          creator: 1 // user
        })
          .then(() => {
            this.$nextTick(function () {
              this.msg = ''
              this.$forceUpdate()
              this.scrollToBottom()
            })
          })
      }
      if (this.isCollapsedAdmin) {
        this.setUnread('unreadByAdmin', this.unreadByAdmin + 1)
      }
    },
    detectTyping () {
      this.isTyping = true
      this.setTyping()
      setTimeout(() => {
        this.isTyping = false
        this.setTyping()
      }, 4000)
    },
    setTyping () {
      this.$store.dispatch('setChatProp', {
        chatId: this.chatId,
        props: 'isTypingUser',
        value: this.isTyping ? 1 : 0
      })
    },
    setCollapse () {
      this.$store.dispatch('setChatProp', {
        chatId: this.chatId,
        props: 'isCollapsedUser',
        value: this.isCollapsedChat ? 1 : 0
      })
    },
    setUnread (by, count) {
      this.$store.dispatch('setChatProp', {
        chatId: this.chatId,
        props: by,
        value: count
      })
    },
    scrollToBottom () {
      if (this.$refs.chatMessages) {
        let chat = this.$refs.chatMessages
        chat.scrollTop = chat.scrollHeight
      }
      if (this.$refs.msgInput) {
        this.$refs.msgInput.focus()
      }
    }
  },
  computed: {
    chatMessages () {
      return this.$store.getters.chatMessages ? this.$store.getters.chatMessages : {}
    },
    isTypingAdmin () {
      return this.$store.getters.chatPropByName('isTypingAdmin')
    },
    isCollapsedAdmin () {
      return this.$store.getters.chatPropByName('isCollapsedAdmin')
    },
    unreadByUser () {
      return this.$store.getters.chatPropByName('unreadByUser')
    },
    unreadByAdmin () {
      return this.$store.getters.chatPropByName('unreadByAdmin')
    },
    isOnlineAdmin () {
      return this.$store.getters.isOnlineAdmin
    }
  },
  watch: {
    chatMessages () {
      this.$nextTick(function () {
        this.scrollToBottom()
      })
    }
  }
}
</script>

<style scoped lang="scss">
  #liveChat {
    z-index: 10;
  }

  .chat_header_user {
    margin-bottom: 1px;
    padding-bottom: 12px;
    background: $color-secondary;
  }

  .chat_input {
    padding: 5px;
  }

  .user_chat_messages {
    width: 100%;
    height: 280px;
    overflow: scroll;
  }

  .user_live_chat {
    position: fixed;
    bottom: 30px;
    right: 40px;
    width: 320px;
    height: 400px;
    z-index: 10000;
  }

  .collapsed_chat {
    position: fixed;
    bottom: 30px;
    right: 40px;
    z-index: 10000;
  }

  textarea {
    border: 1px solid lightgrey;
    border-radius: 3px;
  }

  .fade-enter-active, .fade-leave-active {
    transition: opacity .3s;
  }

  .fade-enter, .fade-leave-to /* .fade-leave-active до версии 2.1.8 */
  {
    opacity: 0;
  }

  .bounce-enter-active {
    animation: bounce-in .5s;
  }

  .bounce-leave-active {
    animation: bounce-in .5s reverse;
  }

  @keyframes bounce-in {
    0% {
      transform: scale(0);
    }
    50% {
      transform: scale(1.1);
    }
    100% {
      transform: scale(1);
    }
  }

  .closeChat {
    margin: 0;
    padding: 0;
  }

  .chat_msg_meta {
    font-size: 10px
  }

  .chat_msg {
    white-space: pre-wrap;
    text-align: left;
  }

  /* User live chat heart */
  .heart {
    width: 64px;
    height: 64px;
    /** height is required as absolute value **/
    background-color: $color-secondary;
    border-radius: 100%;
    position: relative;
    animation: pulse 2000ms linear infinite;
    -webkit-animation: pulse 2000ms linear infinite;
    -moz-animation: pulse 2000ms linear infinite;
  }

  .heart:hover {
    cursor: pointer;
  }

  .heart img {
    height: 27px;
    position: absolute;
    top: 20px;
    left: 16px;
    color: white;
    text-shadow: $primary-text-shadow;
  }

  .unread_by_user {
    position: absolute;
    top: 16px;
    left: 20px;
    font-size: 22px;
    color: white;
  }

  .heart:after,
  .heart:before {
    display: block;
    margin: auto;
    position: absolute;
    content: "";
    width: 64px;
    height: 64px;
    border-radius: 100%;
    background-color: $color-secondary;
  }

  .heart:after {
    z-index: -100;
    -webkit-animation: outer-ripple 2000ms linear infinite;
    -moz-animation: outer-ripple 2000ms linear infinite;
    animation: outer-ripple 2000ms linear infinite;
  }

  .heart:before {
    z-index: -200;
    -webkit-animation: inner-ripple 2000ms linear infinite;
    -moz-animation: inner-ripple 2000ms linear infinite;
    animation: inner-ripple 2000ms linear infinite;
  }

  /* outer ripple */

  @keyframes pulse {
    0% {
      transform: scale(0.8);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
    10% {
      transform: scale(1.1);
      filter: alpha(opacity=1);
      opacity: 1;
    }
    20% {
      transform: scale(0.9);
      filter: alpha(opacity=1);
      opacity: 1;
    }
    100% {
      transform: scale(0.8);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
  }

  @-moz-keyframes pulse {
    0% {
      transform: scale(0.8);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
    10% {
      transform: scale(1.1);
      filter: alpha(opacity=1);
      opacity: 1;
    }
    20% {
      transform: scale(0.9);
      filter: alpha(opacity=1);
      opacity: 1;
    }
    100% {
      transform: scale(0.8);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
  }

  @-webkit-keyframes pulse {
    0% {
      transform: scale(0.8);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
    10% {
      transform: scale(1.1);
      filter: alpha(opacity=1);
      opacity: 1;
    }
    20% {
      transform: scale(0.9);
      filter: alpha(opacity=1);
      opacity: 1;
    }
    100% {
      transform: scale(0.8);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
  }

  @keyframes outer-ripple {
    0% {
      transform: scale(1);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
    80% {
      transform: scale(3);
      filter: alpha(opacity=0);
      opacity: 0;
    }
    100% {
      transform: scale(3);
      filter: alpha(opacity=0);
      opacity: 0;
    }
  }

  @-webkit-keyframes outer-ripple {
    0% {
      transform: scale(1);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
    80% {
      transform: scale(3);
      filter: alpha(opacity=0);
      opacity: 0;
    }
    100% {
      transform: scale(3);
      filter: alpha(opacity=0);
      opacity: 0;
    }
  }

  @-moz-keyframes outer-ripple {
    0% {
      transform: scale(1);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
    80% {
      transform: scale(3);
      filter: alpha(opacity=0);
      opacity: 0;
    }
    100% {
      transform: scale(3);
      filter: alpha(opacity=0);
      opacity: 0;
    }
  }

  /* inner ripple */

  @keyframes inner-ripple {
    0% {
      transform: scale(1);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
    30% {
      transform: scale(1);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
    100% {
      transform: scale(2.5);
      filter: alpha(opacity=0);
      opacity: 0;
    }
  }

  @-webkit-keyframes inner-ripple {
    0% {
      transform: scale(1);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
    30% {
      transform: scale(1);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
    100% {
      transform: scale(2.5);
      filter: alpha(opacity=0);
      opacity: 0;
    }
  }

  @-moz-keyframes inner-ripple {
    0% {
      transform: scale(1);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
    30% {
      transform: scale(1);
      filter: alpha(opacity=50);
      opacity: 0.5;
    }
    100% {
      transform: scale(2.5);
      filter: alpha(opacity=0);
      opacity: 0;
    }
  }
</style>