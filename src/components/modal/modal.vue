
<template>
<transition name="fade">
<div class="dialog" v-if="isShow">
   <div class="mask" ></div>
   <div class="dialog-content" :style="modalWidth">

     <slot></slot>
       <!--<div class="btn-group">
         <div class="btn" @click="handleCancel" >{{cancelText}}</div>
         <div class="btn" @click="handleSure">{{okText}}</div>
      </div>-->
    </div>
</div>
</transition>
</template>
<script>
/**
 * 模态框组件
 * @param isShow 模态框的显示隐藏
 * @param onCancel 模态框回调
 * @param width 模态框宽度配置
 * @param title 模态框主题title配置
 * @param slot 模态框内容填充
 * 引用方式  <modal title="模态框" :isShow="isShow" :onCancel="cancelCb" :width="500" @close="isShow=false">
                <p>这就是文本框内容</p>
            </modal>
 */
export default {
  props: {
    isShow: {type: Boolean,
        default: false

    },
    width: {
      type: Number,
      default: 640
    },
    title: {
      type: String
    },
    okText: {
      type: String,
      default: "确定"
    },
    cancelText: {
      type: String,
      default: "取消"
    },
    onCancel: {
      type: Function,
      default() {}
    },
    onOk: {
      type: Function,
      default() {}
    }
  },
  methods: {
    handleCancel() {
      this.onCancel();
      this.handleClose();
    },
    handleSure() {
      this.onCancel();
      this.handleClose();
    },
    handleClose() {
      this.$emit("close");
    }
  },
  computed: {
    modalWidth() {
      if (this.width !== 640 && this.width !== 0) {
        return { width: `${this.width}px` };
      }
      return null;
    }
  },
  beforeDestroy() {
    this.$el.remove();
  }
};
</script>

<style lang="scss" scoped>
.dialog {
    border-radius: 8px;
  .dialog-content {
    position: fixed;
    box-sizing: border-box;
    width: 600px;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    border-radius: 10px;
    background: #fff;
    z-index: 500002;
    .header{
        height: 56px;
        background: #f8f8f8;
        border-radius: 8px 8px 0 0;
        line-height: 56px;
        .close {
            float: right;
            font-size: 12px;
            cursor: pointer;
            padding-right: 20px;
            img{
                padding-top: 20px;
            }
        }
        .title {
            display: inline-block;
            font-size: 16px;
            font-weight: normal;
            line-height: 30px;
            padding-left: 40px;
            color: #61687c;
        }
    }
    .text {
      font-size: 14px;
      line-height: 30px;
      color: #555;
    }
    .btn-group {
      display: flex;
      position: absolute;
      right: 0;
      bottom: 10px;
      .btn {
        cursor: pointer;
        padding: 10px 20px;
        font-size: 14px;
        &:last-child {
          color: #76d49b;
        }
      }
    }
    .dialog-main {
      font-size: 14px;
    }
  }
  .mask {
    position: fixed;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    z-index: 50001;
    background: rgba(0, 0, 0, 0.5);
  }
}
</style>
