<template>
  <transition @after-leave="afterLeave">
    <div class="ios-alertview-overlay" v-if="showModal">
      <div class="ios-alertview">
        <div class="ios-alertview-inner" :class="{'ios-alertview-inner-remind': !buttons || !buttons.length}">
          <div class="ios-alertview-title" v-if="title">{{ title }}</div>
          <div class="ios-alertview-text" v-html="text" v-if="text"></div>
          <input autofocus class="ios-alertview-text-input" :placeholder="placeholder" v-model="value" v-if="input" />
        </div>
        <div class="ios-alertview-buttons" v-if="buttons && buttons.length" :class="{'ios-alertview-buttons-horizontal': buttons.length <= 2}">
          <span class="ios-alertview-button" :class="{'ios-alertview-button-bold': button.bold}" v-for="(button, index) in buttons" @click.prevent.stop="onClick(button, index)">{{ button.text }}</span>
        </div>
      </div>
    </div>
  </transition>
</template>

<style>
.ios-alertview-overlay{
  font-family: Helvetica Neue,Helvetica,Arial,sans-serif;
  color: #000;
  font-size: 14px;
  line-height: 1.4;
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.4);
  z-index: 10600;
}
.ios-alertview-overlay.v-enter{
  opacity: 0;
}
.ios-alertview-overlay.v-enter-active{
  transition: opacity .4s;
}
 .ios-alertview-overlay.v-leave-active{
   opacity: 0;
   transition: opacity .4s;
 }
.ios-alertview-overlay *{
  -webkit-tap-highlight-color: transparent;
  -webkit-touch-callout: none;
}
.ios-alertview{
  width: 270px;
  position: absolute;
  z-index: 11000;
  left: 50%;
  top: 50%;
  text-align: center;
  border-radius: 7px;
  transform: translate3d(-50%, -50%, 0);
}
.ios-alertview-overlay.v-enter .ios-alertview{
  opacity: 0;
  transform: translate3d(-50%, -50%, 0) scale(1.185);
}
.ios-alertview-overlay.v-enter-active .ios-alertview{
  transition: all .4s;
}
.ios-alertview-overlay.v-leave-active .ios-alertview{
  opacity: 0;
  z-index: 10999;
  transform: translate3d(-50%, -50%, 0) scale(0.815);
  transition: all .4s;
}

.ios-alertview-inner{
  padding: 15px;
  border-bottom: 1px solid #b5b5b5;
  border-top-left-radius: 7px;
  border-top-right-radius: 7px;
  background: #e8e8e8;
}
.ios-alertview-inner-remind{
  border-bottom-right-radius: 7px;
  border-bottom-left-radius: 7px;
}
.ios-alertview-title{
  font-weight: bold;
  font-size: 18px;
  text-align: center;
}
.ios-alertview-title + .ios-alertview-text{
  margin-top: 5px;
}
.ios-alertview-text-input{
  outline: 0;
  box-sizing: border-box;
  height: 30px;
  background: #fff;
  margin: 0;
  margin-top: 15px;
  padding: 0 5px;
  border: 1px solid #a0a0a0;
  border-radius: 5px;
  width: 100%;
  font-size: 14px;
  font-family: inherit;
  display: block;
  box-shadow: 0 0 0 rgba(0, 0, 0, 0);
  appearance: none;
}
.ios-alertview-buttons{
  overflow: hidden;
}
.ios-alertview-buttons-horizontal{
  height: 44px;
  display: flex;
  justify-content: center;
}
.ios-alertview-button{
  width: 100%;
  padding: 0 5px;
  height: 44px;
  font-size: 17px;
  line-height: 44px;
  text-align: center;
  color: #007aff;
  background: #e8e8e8;
  display: block;
  position: relative;
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
  cursor: pointer;
  border-bottom: 1px solid #b5b5b5;
  box-sizing: border-box;
}
.ios-alertview-button.ios-alertview-button-bold{
  font-weight: bold;
}

.ios-alertview-button:active{
  background-color: #d4d4d4;
}
.ios-alertview-button:last-child{
  border-radius: 0 0 7px 7px;
}
.ios-alertview-buttons-horizontal .ios-alertview-button{
  border-bottom: none;
  border-right: 1px solid #b5b5b5;
  flex: 1;
}
.ios-alertview-buttons-horizontal .ios-alertview-button:first-child{ /* cancel */
  border-radius: 0 0 0 7px;
}
.ios-alertview-buttons-horizontal .ios-alertview-button:last-child{ /* OK */
  border-right: none;
  border-radius: 0 0 7px 0;
}
.ios-alertview-buttons-horizontal .ios-alertview-button:first-child:last-child{
  border-radius: 0 0 7px 7px;
}
</style>

<script>

import defer from './defer.js';

// css 动画执行的时间
const ANIMATION_TIME = 400;

export default {
  data: function () {
    return {
      value: '',
      showModal: false
    };
  },
  props: [
    'title',
    'text',
    'input',
    'placeholder',
    'onClick',
    'remindDuration',
    'buttons'
  ],
  methods: {
    activate: function(){
      this._deferred = defer();
      this.showModal = true;

      // no buttons, remind (ANIMATION_TIME + remindDuration) time, then auto remove
      if(!this.buttons || !this.buttons.length){
        setTimeout(() => {
          this.showModal = false;
          this._deferred.resolve();
        }, ANIMATION_TIME + this.remindDuration);
      }

      return this._deferred.promise;
    },
    onClick: function (button, index) {
      var cbkData = {
        index: index,
        button: button,
        value: this.value
      };

      if (typeof button.onClick === 'function') {
        button.onClick(cbkData);
      }

      this._deferred.resolve(cbkData);
      this.showModal = false;
    },
    afterLeave: function () {
      this.$destroy();
      this.$el.parentNode.removeChild(this.$el);
    }
  }
}
</script>
