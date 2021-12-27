<template>
  <v-card class="card-container">    
    <v-card-text v-if="responseCard.subTitle">
      <span>{{responseCard.subTitle}}</span>
    </v-card-text>
    <v-card-media
      v-if="responseCard.imageUrl && !isCarousel"
      v-bind:src="responseCard.imageUrl"
      contain
      height="50vh"
    ></v-card-media>
    <v-card-actions v-if="responseCard.buttons" class="button-row">
      <div class="btns-containers">        
        <button 
          v-for="(button) in responseCard.buttons"
          v-show="button.text && button.value"
          v-bind:key="button.id"
          v-on:click="onButtonClick(button.value)"
          v-bind:disabled="shouldDisableClickedResponseCardButtons"
          round
          default
          class="btns">
          {{button.text}}
        </button>
      </div>
    </v-card-actions>
    <v-card-actions v-if="responseCard.attachmentLinkUrl">
      <v-btn
        flat
        class="red lighten-5"
        tag="a"
        v-bind:href="responseCard.attachmentLinkUrl"
        target="_blank"
      >
        Ingresa aquí
      </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
/*
Copyright 2017-2019 Amazon.com, Inc. or its affiliates. All Rights Reserved.

Licensed under the Amazon Software License (the "License"). You may not use this file
except in compliance with the License. A copy of the License is located at

http://aws.amazon.com/asl/

or in the "license" file accompanying this file. This file is distributed on an "AS IS"
BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, express or implied. See the
License for the specific language governing permissions and limitations under the License.
*/
export default {
  name: 'response-card',
  props: ['response-card', 'is-carousel'],
  data() {
    return {
      hasButtonBeenClicked: false,
    };
  },
  computed: {
    shouldDisplayResponseCardTitle() {
      return this.$store.state.config.ui.shouldDisplayResponseCardTitle;
    },
    shouldDisableClickedResponseCardButtons() {
      return (
        this.$store.state.config.ui.shouldDisableClickedResponseCardButtons &&
        this.hasButtonBeenClicked
      );
    },
  },
  methods: {
    onButtonClick(value) {
      this.hasButtonBeenClicked = true;

      const messageType = this.$store.state.config.ui.hideButtonMessageBubble ? 'button' : 'human';
      const message = {
        type: messageType,
        text: value,
      };

      this.$store.dispatch('postTextMessage', message);
    },
  },
};
</script>

<style scoped>
.card {
  width: 75vw;
  position: inherit; /* workaround to card being displayed on top of toolbar shadow */
  padding-bottom: 0.5em;
  box-shadow: none !important;
  background-color: unset !important;
}

.card__title {
  padding: 0.5em;
  padding-top: 0.75em;
}
.card__text {
  padding: 0.33em;
}

.card__actions .btn {
  margin: 4px 4px;
  font-size: 1em;
  min-width: 44px;
}

.card__actions.button-row {
  justify-content: center;
  padding-bottom: 0.15em;
}

.btns-containers {
  display:flex;
  flex-direction:column;
  width:100%;
}

.btns {  
  align-items: center;
  border: 1px solid #e6e5eb;
  border-radius: 20px;
  color: #4a90e2;
  cursor: pointer;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: center;
  -ms-flex-pack: center;
  justify-content: center;
  padding: 8px 10px;
  margin-bottom: 5px;
}

.btns:hover {
  background: #eaeaea;
  color: #277adc;
}    
</style>
