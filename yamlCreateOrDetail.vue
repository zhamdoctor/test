<!--
 * @Author: fanjf
 * @Date: 2020-06-09 13:22:27
 * @LastEditors: 24min
 * @LastEditTime: 2020-12-11 14:13:55
 * @Description: yaml创建 查看组件
-->
<template>
  <div :class="{ 'main-content': isCreate() ? true : false }">
    <div
      class="content-center"
      :style="mainContentStyle()"
      style="padding:20px;"
    >
      <div class="detail-header" v-if="isCreate()">
        <div class="detail-return-list" @click="returnList">返回列表</div>
        <span class="detail-title">YAML创建</span>
      </div>
      <div
        v-if="type === 'storage' || type === 'volume'"
        :class="`${isCreate() ? 'label-select-create' : 'label-select-detail'}`"
      >
        <hx-label
          style="display:inline-block;"
          label="所属集群"
          :req="isCreate() ? true : false"
        ></hx-label>
        <template v-if="isCreate()">
          <el-select v-model="clusterUuid" placeholder="请选择">
            <el-option
              v-for="item in clusterListMX"
              :key="item.uuid"
              :label="item.name"
              :value="item.uuid"
            ></el-option>
          </el-select>
        </template>
        <template v-else>
          <span>{{ clusterName }}</span>
        </template>
      </div>
      <div class="editor-container">
        <div class="editor-header">
          {{ isCreate() ? "YAML" : "YAML(只读)" }}
        </div>
        <yaml-editor v-model="value" :readOnly="!isCreate()" />
      </div>
      <div class="yaml-creater-footer" v-if="isCreate()">
        <hx-button-group>
          <hx-button @click="todo()">确 定</hx-button>
          <hx-button @click="returnList">取 消</hx-button>
        </hx-button-group>
      </div>
    </div>
  </div>
</template>

<script>
import yamlEditor from "./YamlEditor";
import { clusterList } from "@/common/mixin";
const yamlDeafault =
  '- hosts: all\n  become: yes\n  become_method: sudo\n  gather_facts: no\n\n  tasks:\n  - name: "install {{ package_name1 }}"\n    package:\n      name: "{{ package_name1 }}"\n      state: "{{ state | default(\'present\') }}"';
export default {
  name: "yamlCreateOrDetail",
  props: {
    showType: {
      //是新建可编辑 还是 查看的时候  不可编辑
      type: String,
      default: "create"
    },
    yamlData: {
      type: String,
      default: ""
    },
    clusterName: {
      type: String,
      default: ""
    }
  },
  mixins: [clusterList],
  data() {
    return {
      value: "",
      type: "",
      clusterUuid: ""
    };
  },
  components: { yamlEditor },
  created() {
    this.type = this.$route.params.type;
    this.value = this.yamlData ? jsyaml.dump(JSON.parse(this.yamlData)) : "";
  },
  methods: {
    todo() {
      if (this.value) {
        //判断是否输入yaml
        // if (this.parseYamlFn(this.value).isYaml) {
        if (!this.parseYamlFn(this.value).errorMessage) {
          // this.hxAlert("功能尚在开发中😐");
          console.log(
            JSON.parse(JSON.stringify(jsyaml.load(this.value), null, 4))
          );
          let yamlData = JSON.parse(
            JSON.stringify(jsyaml.load(this.value), null, 4)
          );
          this.$emit("getYamlData", { data: yamlData, uuid: this.clusterUuid });
        } else {
          this.hxAlertConfirm(
            this.parseYamlFn(this.value).errorMessage,
            "提示信息"
          );
        }
      } else {
        this.hxAlert("请输入YAML！", "提示信息");
      }
    },
    isCreate() {
      return this.showType === "create";
    },
    returnList() {
      // this.$router.push({ path: `/${this.type}` });
      this.$router.go(-1);
    },
    parseYamlFn(str) {
      let isYaml = false;
      let errorMessage = "";
      try {
        isYaml = !!jsyaml.load(str);
      } catch (e) {
        errorMessage = e && e.message;
      }
      console.log("yaml",isYaml,errorMessage)
      return {
        isYaml,
        errorMessage
      };
    }
  }
};
</script>

<style scoped lang="scss">
.editor-container {
  margin-top: 15px;
  position: relative;
  height: 100%;
  & .editor-header {
    padding-left: 15px;
    border: 1px solid #e5e6e6;
    height: 36px;
    line-height: 36px;
    font-weight: bold;
    font-size: 16px;
  }
}
.yaml-creater-footer {
  text-align: center;
  height: 40px;
  padding-top: 15px;
}
.label-select-create {
  margin: 10px 0px;
}
.label-select-detail {
}
</style>
