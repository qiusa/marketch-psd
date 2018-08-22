<template>
  <transition name='fade'>
    <div v-show='visible' :class='position' class='toast'>
      <i>{{message}}</i>
    </div>
  </transition>
</template>
<script>
  import Vue from 'vue'
  export default Toast{
    data() {
      return {
        visible: false,
        message: '',
        position: ''
      };
    },
    mounted() {
      var instance = new ToastConstructor().$mount(document.createElement('div'))
      let duration = options.duration || 5000;
      instance.message = typeof options === 'string' ? options : options.message
      instance.position = options.position || 'middle'
      document.body.appendChild(instance.$el);
      instance.visible = true;
      Vue.nextTick(() => {
        instance.timer = setTimeout(function () {
          instance.close();
        }, duration);
      })
      return instance
    },
    methods: {
      close() {
        let removeDom = event => {
          event.target.parentNode.removeChild(event.target);
        };
        this.visible = false;
        this.$el.addEventListener('transitionend', removeDom);
      }
    }
  }

</script>
<style lang="less">
  .toast {
    position: fixed;
    left: 50%;
    -webkit-transform: translate(-50%, -50%) scale(1);
    -ms-transform: translate(-50%, -50%) scale(1);
    -o-transform: translate(-50%, -50%) scale(1);
    transform: translate(-50%, -50%) scale(1);
    word-wrap: break-word;
    padding: 10px;
    text-align: center;
    z-index: 10002;
    font-size: .3rem;
    max-width: 80%;
    color: #fff;
    border-radius: 5px;
    background: rgba(0, 0, 0, 0.7);
    overflow: hidden;
    i {
      color: #fff;
    }
  }

  .toast.middle {
    top: 50%;
  }

  .toast.top {
    top: 10%;
  }

  .toast.bottom {
    top: 90%;
  }

  .fade-enter-active,
  .fade-leave-active {
    -webkit-transition: transform 0.5s;
    -o-transition: transform 0.5s;
    transition: transform 0.5s;

  }

  .fade-enter,
  .fade-leave-active {
    -webkit-transform: translate(-50%, -50%) scale(0);
    -ms-transform: translate(-50%, -50%) scale(0);
    -o-transform: translate(-50%, -50%) scale(0);
    transform: translate(-50%, -50%) scale(0);
  }
</style>