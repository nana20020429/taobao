<template>
  <div class="s1 msg-code">
    <!-- prop="code" -->
    <el-form-item label="短信验证码">
      <el-row>
        <el-col :span="16">
          <el-input id="sendCode-input" @blur="testCode" v-model="code" placeholder="请输入验证码"></el-input>
        </el-col>
        <el-col :span="8">
          <el-button id="code" type="primary" :disabled="isDisabled" @click="send">{{buttonText}}</el-button>
        </el-col>
      </el-row>
    </el-form-item>
    <van-popup v-model="show" @click-overlay="clickOverlay()">
      <drag-verify
        ref="dragVerify"
        completedBg="#00B7D3"
        progressBarBg="#00B7D3"
        :isPassing="isPassing"
        @passcallback="passcallback()"
        text="请按住滑块，拖动到最右边"
        successText="验证通过"
        handlerIcon="el-icon-d-arrow-right"
        successIcon="el-icon-circle-check"
      />
    </van-popup>
  </div>
</template>

<script>
import DragVerify from "components/common/dragVerify/DragVerify"; //滑动验证
import { Toast } from "vant";
export default {
  name: "SMSVerification",
  components: {
    DragVerify,
  },
  data() {
    return {
      isPassing: false, //是否验证通过
      show: false, //是否展示验证码验证
      buttonText: "发送验证码",
      isDisabled: false, // 是否禁止点击发送验证码按钮
      //标识是否发送短信
      flag: false,
      code: "",
      //从服务器拿到的验证码
      authCode: "",
    };
  },
  //codeIsOK 验证是否通过 默认为false
  props: ["phone", "codeIsOK", "codeType", "phoneIsOK", "type"],
  methods: {
    // 点击发送按钮
    send() {
      this.show = true;
    },
    // 滑块验证成功
    passcallback() {
      this.isPassing = true;
      this.show = false;
      let tel = this.phone;
      if (this.checkMobile(tel)) {
        this.sendCode();
      }
    },
    // 点击遮罩层时触发
    clickOverlay() {
      this.isPassing = false; //设置可滑动
      // 重置滑块移动量
      this.$refs.dragVerify.reset();
    },
    // <!--发送验证码-->
    async sendCode() {
      let tel = this.phone;
      // 验证通过 可进行验证码的发送
      Toast.loading({
        message: "发送中...",
        forbidClick: true,
      });
      await this.$api.code
        .getCode({
          phone: this.phone,
          codeType: this.codeType,
        })
        .then((res) => {
          Toast.clear();
          // console.log(res);
          if (res.data.code != 200) {
            this.isPassing = false; //设置可滑动
            // 重置滑块移动量
            this.$refs.dragVerify.reset();
            return Toast.fail(res.data.msg);
          }
          // 如果发送成功 取出服务器返回的验证码
          this.authCode = res.data.msg;
          Toast.success("发送成功");
          this.sendOk();
        })
        .catch((err) => {
          Toast.clear();
          this.isPassing = false; //设置可滑动
          // 重置滑块移动量
          this.$refs.dragVerify.reset();
          return Toast.fail("验证码发送异常，请稍后再试");
        });
    },
    sendOk() {
      let time = 60;
      this.buttonText = "已发送";
      this.isDisabled = true;
      this.flag = true;
      if (this.flag) {
        this.flag = false;
        let timer = setInterval(() => {
          time--;
          this.buttonText = time + " 秒";
          if (time === 0) {
            clearInterval(timer);
            this.buttonText = "重新获取";
            this.isDisabled = false;
            this.flag = false;
          }
        }, 1000);
      }
    },
    // 验证手机号
    checkMobile(str) {
      let re = /^(?:(?:\+|00)86)?1(?:(?:3[\d])|(?:4[5-7|9])|(?:5[0-3|5-9])|(?:6[5-7])|(?:7[0-8])|(?:8[\d])|(?:9[1|8|9]))\d{8}$/;
      if (re.test(str)) {
        return true;
      } else {
        this.isPassing = false; //设置可滑动
        // 重置滑块移动量
        this.$refs.dragVerify.reset();
        Toast.fail("请输入之正确的手机号");
        return false;
      }
    },
    //验证码校验
    testCode() {
      //console.log(this.code);
      if (this.authCode === null) {
        return this.$message.error("请输入验证码！");
      }
      //隐式转换下 保证下统一类型
      if (this.authCode - 0 !== this.code - 0) {
        return this.$message.error("验证码不正确！");
      }
      //发送一个事件出去 提示已完成验证
      this.$emit("codeOk");
    },
  },
};
</script>

<style>
#code {
  margin-left: 5px;
}
</style>