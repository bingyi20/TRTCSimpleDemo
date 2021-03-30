<template name="board">
  <div id="board">

  </div>
</template>

<script>
import TIM from 'tim-js-sdk';
import COS from "core-js";
import axios from "axios"; 
import TEduBoard from '../libs/TEduBoard.256';
import genTestUserSig from '../debug/gen-test-user-sig';

window.COS = COS;
window.axios = axios;

export default {
  props: ['roomId', 'userId'],
  data() {
    return {
      teduBoard: null,
      sdkInfo: null,
      tim: null
    }
  },
  mounted() {
    // eslint-disable-next-line no-debugger
    this.sdkInfo = genTestUserSig(this.userId);
    
    this.initTim();

    this.initBoard();
  },

  methods: {
    /**
     * tim初始化
     */
    initTim() {
      let timOptions = {
        sdkAppID: this.sdkInfo.sdkappid
      };
      this.tim = TIM.create(timOptions);
      // 注册 COS SDK 插件
      this.tim.registerPlugin({ 'cos-js-sdk': COS });
      // eslint-disable-next-line no-debugger
      this.timSDKReady().then(() => {
        this.timLogin()
      }).then(() => {
        this.timJoinGroup()
      }).then(() => {
        let message = this.tim.createTextMessage({ to: 'user1', conversationType: 'C2C', payload: { text: 'Hello world!' }});
        this.tim.sendMessage(message);
        console.log('Tim消息发送成功！');
      })
    },

    /**
     * 监听tim初始化
     */
    timSDKReady() {
      return new Promise((resolve) => {
        this.tim.on(TIM.EVENT.SDK_READY, () => {
          resolve();
        }, this);
      });
    },

    /**
     * tim登录
     */
    timLogin() {
      return this.tim.login({userID: this.sdkInfo.sdkappid, userSig: this.sdkInfo.userSig});
    },

    /**
     * 加入群组
     */
    timJoinGroup() {
      return this.tim.joinGroup({ groupID: this.roomId, type: TIM.TYPES.GRP_AVCHATROOM });
    },

    /**
     * 初始化白板
     */
    initBoard() {
      var initParams = {
        id: 'board',
        classId: this.roomId,
        sdkAppId: this.sdkInfo.sdkappid,
        userId: this.userId,
        userSig: this.sdkInfo.userSig
      };
      this.teduBoard = new TEduBoard(initParams);
      // 初始化完成
      this.teduBoard.on(TEduBoard.EVENT.TEB_INIT, () => {
        // 设置宽高比
        this.teduBoard.setBoardRatio('11:5');
        // 设置背景色
        this.teduBoard.setGlobalBackgroundColor('rgba(225, 225, 225, 0)');
        // 设置光标渲染模式为原生渲染光标(更加流畅)
        this.teduBoard.setSystemCursorEnable(true);
      });
      // 监控白板报错
      this.teduBoard.on(TEduBoard.EVENT.TEB_ERROR, (code, msg) => {
        console.error(code, msg);
      });
      // 监控白板报警事件
      this.teduBoard.on(TEduBoard.EVENT.TEB_WARNING, (code, msg) => {
        console.warn(code, msg);
      });
    }
  },

  /**
   * 组件销毁的时候，将白板进行销毁
   */
  beforeDestroy() {
    this.teduBoard.destroy();
  }
  
}
</script>

<style scoped>
  #board {
    width: 100%;
    height: 100%;
  }
</style>