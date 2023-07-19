<template>
  <el-dialog
    :title="
      id
        ? $t('RULE_ENGINE.DATA_GATEWAY.EDIT')
        : $t('RULE_ENGINE.DATA_GATEWAY.CREATE')
    "
    class="el-dark-dialog"
    :close-on-click-modal="false"
    :before-close="handleClose"
    :visible.sync="dataGatewayDialogVisible"
    width="30%"
    height="60%"
    top="10vh"
  >
    <el-form
      ref="CreateForm"
      label-position="left"
      label-width="140px"
      style="margin-left: 5%"
      :model="form"
    >
      <el-form-item :label="$t('RULE_ENGINE.DATA_GATEWAY.NAME')" required>
        <el-col :span="20">
          <el-input ref="nameRef" v-model="form.name"></el-input>
        </el-col>
      </el-form-item>

      <el-form-item
        :label="$t('RULE_ENGINE.DATA_GATEWAY.SIGNATURE_MODE')"
        required
      >
        <el-col :span="20">
          <el-select
            style="width: 100%"
            class="el-dark-input"
            v-model="form.signature_mode"
          >
            <el-option
              class="el-dark-input"
              v-for="item in signMethodChoice"
              :key="item"
              :label="item"
              :value="item"
            ></el-option>
          </el-select>
        </el-col>
      </el-form-item>

      <el-form-item
        :label="$t('RULE_ENGINE.DATA_GATEWAY.IP_WHITELIST')"
        required
      >
        <el-col :span="20">
          <el-input
            class="el-dark-input"
            type="textarea"
            ref="nameRef"
            rows="4"
            v-model="form.ip_whitelist"
          ></el-input>
        </el-col>
      </el-form-item>

      <el-form-item
        :label="$t('RULE_ENGINE.DATA_GATEWAY.DEVICE_ACCESS_SCOPE')"
        required
      >
        <el-col :span="20">
          <el-select
            style="width: 100%"
            class="el-dark-input"
            v-model="form.device_access_scope"
          >
            <el-option
              class="el-dark"
              v-for="item in deviceAccessScopeChoice"
              :key="item.value"
              :label="item.name"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-col>
      </el-form-item>

      <el-form-item
        :label="$t('RULE_ENGINE.DATA_GATEWAY.API_ACCESS_SCOPE')"
        required
      >
        <el-col :span="20">
          <el-select
            style="width: 100%"
            class="el-dark-input"
            v-model="form.api_access_scope"
          >
            <el-option
              class="el-dark-input"
              v-for="item in apiAccessScopeChoice"
              :key="item.value"
              :label="item.name"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-col>
      </el-form-item>

      <el-form-item :label="$t('RULE_ENGINE.DATA_GATEWAY.DESCRIPTION')">
        <el-col :span="20">
          <el-input
            class="el-dark-input"
            ref="descRef"
            type="textarea"
            v-model="form.description"
          ></el-input>
        </el-col>
      </el-form-item>

      <div style="display: flex; justify-content: center">
        <el-button
          class="cancel-button"
          type="cancel"
          size="medium"
          plain
          @click="cancelDialog"
          >{{ $t("RULE_ENGINE.DATA_GATEWAY.CANCEL") }}</el-button
        >
        <el-button
          style="margin-left: 10px"
          class="medium"
          type="save"
          size="medium"
          @click="onSubmit"
          >{{ $t("RULE_ENGINE.DATA_GATEWAY.SUBMIT") }}</el-button
        >
      </div>
    </el-form>
  </el-dialog>
</template>

<script>
// import options from './cascader_options'
import { message_error } from "@/utils/helpers";
import {
  getTranspondNewAdd,
  getTranspondNewEdit,
  getTranspondNewDetail,
} from "@/api/transpondNew";
// const required = true;
export default {
  name: "CreateForm",
  props: {
    // handle_create:{
    //   request: true,
    //   type: Function,
    // },
    // init_data: {
    //   default: null
    // },
    id: { default: null },
    data: { default: null },
    visible: {
      type: [Boolean],
      default: false,
    },
  },
  computed: {
    dataGatewayDialogVisible: {
      get() {
        return this.visible;
      },
      set(val) {
        this.$emit("update:visible", val);
      },
    },
  },
  watch: {
    visible: {
      handler(newValue) {
        if (newValue && this.id) {
          // 编辑
          this.form = this.data;
        }
      },
    },
    dialogMQTTVisible: {
      handler(val) {
        if (!val) {
          this.editData = "";
        }
      },
      deep: true,
      immediate: true,
    },
    dialogUrlVisible: {
      handler(val) {
        if (!val) {
          this.editData = "";
        }
      },
      deep: true,
      immediate: true,
    },
  },
  data: () => ({
    dialogChooseVisible: false, // 二选一弹框
    dialogMQTTVisible: false,
    dialogUrlVisible: false,
    // 签名方式选项
    signMethodChoice: ["MD5", "AES-256"],
    // 接口访问范围选项
    apiAccessScopeChoice: [
      { name: "全部", value: 1 },
      { name: "部分", value: 2 },
    ],
    // 设备访问范围选项
    deviceAccessScopeChoice: [
      { name: "全部", value: 1 },
      { name: "部分", value: 2 },
    ],
    // 列表数据
    listData: [],
    isShow: false,
    // 选中编辑数据
    editData: "",
    // 数据类型
    type: "",
    isChoose: null,
    form: {
      id: "",
      name: "",
      app_key: "",
      signature_mode: "",
      ip_whitelist: "",
      device_access_scope: 1,
      api_access_scope: 1,
      created_at: 0,
      tenant_id: "",
      description: "",
    },
    // rules: {
    //   name: [
    //     {required, message: i18n.t('RULE_ENGINE.DATA_FORWARDINGNEW.PLACEHOLDER')}
    //   ],
    //   desc: [
    //     {required, message: i18n.t('RULE_ENGINE.DATA_FORWARDINGNEW.PLACEHOLDER1')}
    //   ]
    // },
  }),
  created() {
    // if(this.init_data){
    //   this.form.name = this.init_data.name
    // }
  },
  methods: {
    // mq表提交
    create(data) {
      if (this.type == "add") {
        data.title = "推送到外部 MQTT";
        this.listData.push(data);
        this.form.target_info.mqtt = data;
        this.dialogMQTTVisible = false;
      } else {
        data.title = "推送到外部 MQTT";
        this.listData[this.editIndex] = data;
        this.form.target_info.mqtt = data;
        this.dialogMQTTVisible = false;
      }
    },
    // url表提交
    create2(item) {
      if (this.type == "add") {
        item.title = "推送到外部 URL";
        this.listData.push(item);
        this.form.target_info.url = item.url;
        this.dialogUrlVisible = false;
      } else {
        item.title = "推送到外部 URL";
        this.listData[this.editIndex] = item;
        this.form.target_info.url = item.url;
        this.dialogUrlVisible = false;
      }
    },
    // 逻辑删除
    deleteBtn(index) {
      this.listData.splice(index, 1);
    },
    // 编辑
    editBtn(e, index) {
      this.editData = e;
      this.editIndex = index;
      this.type = "edit";
      switch (e.title) {
        case "推送到外部 MQTT":
          this.dialogMQTTVisible = true;
          this.$nextTick(() => {
            this.$refs.realForm.edit(e);
          });
          break;
        case "推送到外部 URL":
          this.dialogUrlVisible = true;
          this.$nextTick(() => {
            this.$refs.realForms.edit(e);
          });
          break;
      }
    },
    /**
     * @description: 新增一行命令
     * @return {*}
     */
    handleAddCommand() {
      this.form.commands.push({ data: {} });
    },
    /**
     * @description: 指定行被改变
     * @param {*} command  改变前的值
     * @param {*} v  改变后的值
     * @return {*}
     */
    handleCommandChange(command, v) {
      command.data = v;
    },
    /**
     * @description: 删除指定行
     * @param {*} command
     * @param {*} v
     * @return {*}
     */
    handleDeleteCommand(command) {
      let index = this.form.commands.findIndex((item) => item == command);
      this.form.commands.splice(index, 1);
    },

    handleAdd() {
      console.log(this.listData, "this");
      if (this.listData.length > 0) {
        this.$message({
          message: "目前只支持新增一条",
          center: true,
          type: "waring",
        });
        this.dialogChooseVisible = false;
        return;
      } else {
        this.dialogChooseVisible = true;
      }
    },

    //二选一弹框
    choose(type) {
      this.editIndex = "";
      if (type == 0) {
        this.type = "add";
        this.isChoose = 0;
        this.dialogChooseVisible = false;
        this.dialogMQTTVisible = true;
      } else {
        this.type = "add";
        this.isChoose = 1;
        this.dialogChooseVisible = false;
        this.dialogUrlVisible = true;
      }
    },

    //详情
    getDetail(id) {
      this.listData = [];
      getTranspondNewDetail({ data_transpond_id: id }).then((res) => {
        if (res.data.code === 200) {
          let data = res.data.data;
          if (data !== "{}") {
            let commands = this.getCommands(JSON.parse(JSON.stringify(data)));
            let tmp = JSON.parse(JSON.stringify(data));
            tmp.commands = commands;
            this.form = tmp;
            if (data.target_info.url !== "") {
              data.title = "推送到外部 URL";
              this.listData.push(data);
            } else {
              data.title = "推送到外部 MQTT";
              this.listData.push(data);
            }
          }
        }
      });
    },

    getCommands(v) {
      let cmds = v?.device_info || [];
      let commands = cmds.map((cmd) => {
        return {
          data: {
            projectId: cmd.business_id,
            groupId: cmd.asset_group_id,
            // deviceId: cmd.device_id,
            device: cmd.device_id,
            messageType: cmd.message_type,
          },
        };
      });
      return commands;
    },

    //提交
    onSubmit() {
      if (!this.validate()) {
        return;
      }

      let params = JSON.parse(JSON.stringify(this.form));

      params.device_info = this.form.commands.map((cmd) => {
        return {
          business_id: cmd.data.projectId,
          asset_group_id: cmd.data.groupId,
          device_id: cmd.data.device,
          message_type: cmd.data.messageType,
        };
      });

      if (!this.id) {
        if (this.form.target_info.url) {
          params.target_info = {
            mqtt: {
              host: "",
              topic: "",
              password: "",
              username: "",
              client_id: "",
              port: null,
            },
            url: this.form.target_info.url,
          };
        } else {
          params.target_info = {
            mqtt: {
              host: this.form.target_info.mqtt.host,
              topic: this.form.target_info.mqtt.topic,
              password: this.form.target_info.mqtt.password,
              username: this.form.target_info.mqtt.username,
              client_id: this.form.target_info.mqtt.client_id,
              port: Number(this.form.target_info.mqtt.port),
            },
            url: "",
          };
        }
      } else {
        if (params.target_info.url) {
          params.target_info = {
            mqtt: {
              host: "",
              topic: "",
              password: "",
              username: "",
              client_id: "",
              port: null,
            },
            url: params.target_info.url,
          };
        } else {
          params.target_info = {
            mqtt: {
              host: this.form.target_info.mqtt.host,
              topic: this.form.target_info.mqtt.topic,
              password: this.form.target_info.mqtt.password,
              username: this.form.target_info.mqtt.username,
              client_id: this.form.target_info.mqtt.client_id,
              port: Number(this.form.target_info.mqtt.port),
            },
            url: "",
          };
        }
      }

      if (!this.id) {
        getTranspondNewAdd(params).then((res) => {
          if (res.data.code === 200) {
            this.$emit("submit");
            this.listData = [];
            this.dataGatewayDialogVisible = false;
            this.$message({
              message: "新建成功",
              center: true,
              type: "success",
            });
          }
        });
      } else {
        getTranspondNewEdit(params).then((res) => {
          if (res.data.code === 200) {
            this.$emit("submit");
            this.listData = [];
            this.dataGatewayDialogVisible = false;
            this.$message({
              message: "编辑成功",
              center: true,
              type: "success",
            });
          }
        });
      }
    },
    handleClose() {
      this.form = {};
      this.listData = [];
      this.dataGatewayDialogVisible = false;
    },
    cancelDialog() {
      this.form = {};
      this.listData = [];
      this.dataGatewayDialogVisible = false;
    },
    validate() {
      if (!this.form.name || this.form.name === "") {
        this.$refs.nameRef.focus();
        message_error(this.$t("RULE_ENGINE.DATA_FORWARDINGNEW.PLACEHOLDER"));
        return false;
      }
      if (!this.form.desc || this.form.desc === "") {
        this.$refs.descRef.focus();
        message_error(this.$t("RULE_ENGINE.DATA_FORWARDINGNEW.PLACEHOLDER1"));
        return false;
      }
      if (!this.form.script) {
        message_error("解析脚本不能为空");
        return;
      }
      for (let i = 0; i < this.$refs.deviceTypeRef.length; i++) {
        const ref = this.$refs.deviceTypeRef[i];
        if (!ref.validate()) return false;
      }
      return true;
    },
  },
};
</script>

<style scoped lang="scss">
.code-editor-label {
  color: #fff;
  margin-top: 10px;
}
::v-deep .code_editor .code_area textarea {
  overflow-y: auto;
}
.dialog-box {
  border: 1px solid #e9e9eb;
  padding: 10px;
  cursor: pointer;
  p {
    margin-bottom: 0;
  }
}
.dialog-box:last-child {
  margin-top: 20px;
}

.dialog-border {
  border: 1px solid #5867dd;
  padding: 10px;
  cursor: pointer;
  p {
    margin-bottom: 0;
  }
}

.dialog-border:last-child {
  margin-top: 20px;
}

.list_box {
  margin-top: 20px;
  .item {
    display: flex;
    align-items: center;
    height: 60px;
    justify-content: space-between;
    border-bottom: 1px solid #ccc;
    .flex_full {
      flex: 1;
    }
    &:first-child {
      border-top: 1px solid #ccc;
    }
  }
}
</style>
