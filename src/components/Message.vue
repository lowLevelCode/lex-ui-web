<template>
  <div class="the-container">
    <div :class="message.type === 'human' ? 'message-container-human' : 'message-container-bot' ">
          <div
                v-if="shouldShowAvatarImage"
                v-bind:style="avatarBackground"
                tabindex="-1"
                class="avatar"
                aria-hidden="true"
              >
          </div>
              <div
                tabindex="0"
                v-on:focus="onMessageFocus"
                v-on:blur="onMessageBlur"
                class="message-bubble focusable"
              >              
              <div>                
                <message-text
                  v-bind:message="message"
                  v-if="'text' in message && message.text !== null && message.text.length"
                ></message-text>

                <div v-if="message.responseCard && message.responseCard.genericAttachments.length == 1">
                  <response-card
                    v-for="(card, index) in message.responseCard.genericAttachments"
                    :key="index"
                    class="response-card"
                    v-bind:response-card="card"            
                    >
                  </response-card>
                </div>
              </div>                  
          </div>
    </div>
    
      <div class="carousel-container">
        <div class="carousel-content" v-if="message.responseCard && message.responseCard.genericAttachments.length > 1">
          <v-carousel hide-delimiters :interval="100000" v-model="cardDataIndex">
            <v-carousel-item
            v-for="(card, index) in message.responseCard.genericAttachments"
            :key="index"
            :src="card.imageUrl"
            reverse-transition="fade"
            transition="fade"
            >
            <!--
              <response-card
              class="response-card"
              v-bind:response-card="card"            
              >
              </response-card>
            -->. 
            </v-carousel-item>
          </v-carousel>
          <br/>
          <response-card
              v-if="currentCardData"
              class="response-card"
              v-bind:response-card="currentCardData"
              v-bind:is-carousel="true"
              >
          </response-card>        
        </div>
      </div>
  </div>
</template>

<script>
import MessageText from './MessageText';
import ResponseCard from './ResponseCard';

export default {
  name: 'message',
  props: ['message', 'feedback'],
  components: {
    MessageText,
    ResponseCard,
  },
  data() {
    return {
      cardDataIndex:0,
      isMessageFocused: false,
      messageHumanDate: 'Now',
      positiveClick: false,
      negativeClick: false,
      hasButtonBeenClicked: false,
      positiveIntent: this.$store.state.config.ui.positiveFeedbackIntent,
      negativeIntent: this.$store.state.config.ui.negativeFeedbackIntent,
      hideInputFields: this.$store.state.config.ui.hideInputFieldsForButtonResponse,
      items: [
          {
            src: 'https://cdn.vuetifyjs.com/images/carousel/squirrel.jpg'
          },
          {
            src: 'https://cdn.vuetifyjs.com/images/carousel/sky.jpg'
          },
          {
            src: 'https://cdn.vuetifyjs.com/images/carousel/bird.jpg'
          },
          {
            src: 'https://cdn.vuetifyjs.com/images/carousel/planet.jpg'
          }
        ]
    };
  },
  computed: {
    currentCardData() {
      if(this.message.responseCard)
        return this.message.responseCard.genericAttachments[this.cardDataIndex];
    },
    botDialogState() {
      if (!('dialogState' in this.message)) {
        return null;
      }
      switch (this.message.dialogState) {
        case 'Failed':
          return { icon: 'error', color: 'red', state: 'fail' };
        case 'Fulfilled':
        case 'ReadyForFulfillment':
          return { icon: 'done', color: 'green', state: 'ok' };
        default:
          return null;
      }
    },
    isLastMessageFeedback() {
      if (this.$store.state.messages.length > 2 && this.$store.state.messages[this.$store.state.messages.length - 2].type !== 'feedback') {
        return true;
      }
      return false;
    },
    botAvatarUrl() {
      return this.$store.state.config.ui.avatarImageUrl;
    },
    agentAvatarUrl() {
      return this.$store.state.config.ui.agentAvatarImageUrl;
    },
    showDialogStateIcon() {
      return this.$store.state.config.ui.showDialogStateIcon;
    },
    showMessageMenu() {
      return this.$store.state.config.ui.messageMenu;
    },
    showDialogFeedback() {
      if (this.$store.state.config.ui.positiveFeedbackIntent.length > 2
      && this.$store.state.config.ui.negativeFeedbackIntent.length > 2) {
        return true;
      }
      return false;
    },
    showErrorIcon() {
      return this.$store.state.config.ui.showErrorIcon;
    },
    shouldDisplayResponseCard() {
      return (
        this.message.responseCard &&
        (this.message.responseCard.version === '1' ||
         this.message.responseCard.version === 1) &&
        this.message.responseCard.contentType === 'application/vnd.amazonaws.card.generic' &&
        'genericAttachments' in this.message.responseCard &&
        this.message.responseCard.genericAttachments instanceof Array
      );
    },
    shouldShowAvatarImage() {      
      if (this.message.type === 'bot') {
        return this.botAvatarUrl;
      } else if (this.message.type === 'human') {
        return this.agentAvatarUrl;
      }
      return false;
    },
    avatarBackground() {
      const avatarURL = (this.message.type === 'bot') ? this.botAvatarUrl : this.agentAvatarUrl;
      return {
        background: `url(${avatarURL}) center center / contain no-repeat`,
      };
    },
    shouldShowMessageDate() {
      return this.$store.state.config.ui.showMessageDate;
    },
  },
  methods: {
    resendMessage(messageText) {
      const message = {
        type: 'human',
        text: messageText,
      };
      this.$store.dispatch('postTextMessage', message);
    },
    onButtonClick(feedback) {
      if (!this.hasButtonBeenClicked) {
        this.hasButtonBeenClicked = true;
        if (feedback === this.$store.state.config.ui.positiveFeedbackIntent) {
          this.positiveClick = true;
        } else {
          this.negativeClick = true;
        }
        const message = {
          type: 'feedback',
          text: feedback,
        };
        this.$emit('feedbackButton');
        this.$store.dispatch('postTextMessage', message);
      }
    },
    playAudio() {
      // XXX doesn't play in Firefox or Edge
      /* XXX also tried:
      const audio = new Audio(this.message.audio);
      audio.play();
      */
      const audioElem = this.$el.querySelector('audio');
      if (audioElem) {
        audioElem.play();
      }
    },
    onMessageFocus() {
      if (!this.shouldShowMessageDate) {
        return;
      }
      this.messageHumanDate = this.getMessageHumanDate();
      this.isMessageFocused = true;
      if (this.message.id === this.$store.state.messages.length - 1) {
        this.$emit('scrollDown');
      }
    },
    onMessageBlur() {
      if (!this.shouldShowMessageDate) {
        return;
      }
      this.isMessageFocused = false;
    },
    getMessageHumanDate() {
      const dateDiff = Math.round((new Date() - this.message.date) / 1000);
      const secsInHr = 3600;
      const secsInDay = secsInHr * 24;
      if (dateDiff < 60) {
        return 'Now';
      } else if (dateDiff < secsInHr) {
        return `${Math.floor(dateDiff / 60)} min ago`;
      } else if (dateDiff < secsInDay) {
        return this.message.date.toLocaleTimeString();
      }
      return this.message.date.toLocaleString();
    },
  },
  created() {
    if (this.message.responseCard && 'genericAttachments' in this.message.responseCard) {
      if (this.message.responseCard.genericAttachments[0].buttons &&
          this.hideInputFields && !this.$store.state.hasButtons) {
        this.$store.dispatch('toggleHasButtons');
      }
    } else if (this.$store.state.config.ui.hideInputFieldsForButtonResponse) {
      if (this.$store.state.hasButtons) {
        this.$store.dispatch('toggleHasButtons');
      }
    }
  },
};
</script>

<style scoped>
.smicon {
  font-size: 14px;
}

.message, .message-bubble-column {
  flex: 0 0 auto;
}

.message, .message-bubble-row {
  max-width: 80vw;
}

.avatar {
  align-self: center;
  border-radius: 50%;
  min-width: calc(2.5em + 1.5vmin);
  min-height: calc(2.5em + 1.5vmin);
  align-self: flex-start;
  margin-right: 4px;
}

.message-bubble {
  border-radius: 24px;
  display: inline-flex;
  font-size: calc(1em + 0.25vmin);
  padding: 0 12px;
  width: fit-content;
  align-self: center;
}

.focusable {
  box-shadow: 0 0.25px 0.75px rgba(0,0,0,0.12), 0 0.25px 0.5px rgba(0,0,0,0.24);
  transition: all 0.3s cubic-bezier(.25,.8,.25,1);
  cursor: default;
}

.focusable:focus {
  box-shadow: 0 1.25px 3.75px rgba(0,0,0,0.25), 0 1.25px 2.5px rgba(0,0,0,0.22);
  outline: none;
}

.message-bot .message-bubble {
  background-color: #E5F1FF; /* red-50 from material palette */
}

.message-agent .message-bubble {
  background-color: #E5F1FF; /* red-50 from material palette */
}
.message-human .message-bubble {
  background-color: #0099FF; /* indigo-50 from material palette */  
  color:white;
}

.message-feedback .message-bubble {
  background-color: #0099FF;  
  color:white;
}

.dialog-state {
  display: inline-flex;
}

.icon.dialog-state-ok {
  color: green;
}
.icon.dialog-state-fail {
  color: red;
}

.play-icon {
  font-size: 2em;
}

.feedback-state {
  display: inline-flex;
  align-self: center;
}

.icon.feedback-icons-positive{
  color: grey;
  /* color: #E8EAF6; */
  /* color: green; */
  padding: .125em;
}

.positiveClick{
  color: green;
  padding: .125em;
}

.negativeClick{
  color: red;
  padding: .125em;
}

.icon.feedback-icons-positive:hover{
  color:green;
}

.icon.feedback-icons-negative{
  /* color: #E8EAF6; */
  color: grey;
  padding: .125em;
}

.icon.feedback-icons-negative:hover{
  color: red;
}

.response-card {
  justify-content: center;
  width: 85vw;
}

.no-point {
  pointer-events: none;
}

.carousel-content {
  width:100%;  
  padding:12px;
  border:1px solid #ccc;  
  border-radius:8px;
}

.carousel-container {
  padding:8px;
}

.carousel {
  height: 200px !important;
  border-radius:20px;  
}

.v-carousel__next > button,
.v-carousel__prev > button {
 color: red !important; 
}

.response-card {
  width:98%;
}

.message-container-bot {
  display:flex;  
  width:100%;
  padding:12px;
}

.message-container-human {
  display:flex;  
  flex-direction:row-reverse;
  width:100%;
  padding-right:12px;  
}

.the-container {
  width:100%;
}
</style>
