<!--包含yaml和页面端编辑-->
<template>
  <div>
    <el-tabs type="border-card" @tab-click="trigEdit">
      <el-tab-pane><span slot="label"><i class="el-icon-chat-dot-square"/>编辑</span>
<!--        crdForm为item信息-->
        <el-form :model="crdForm" label-width="80px">
          <el-divider>ASTS信息</el-divider>
          <el-form-item label="名称"><el-input v-model="crdForm.metadata.name"></el-input></el-form-item>
          <el-form-item label="命名空间"><el-input v-model="crdForm.metadata.namespace"></el-input></el-form-item>
          <el-form-item label="副本数"><el-input v-model.number="crdForm.spec.replicas"></el-input></el-form-item>
          <el-divider>POD模板</el-divider>
          <el-form-item label="CRD标签">
            <div  v-for="(item,j) in getKey(crdForm.spec.template.metadata.labels)" :key="item">
              <el-row :gutter="1">
                <el-col :span="6">
            <el-input v-model="Object.keys(crdForm.spec.template.metadata.labels)[j]"></el-input>
                </el-col>
                <el-col :span="6">
              <el-input v-model="crdForm.spec.template.metadata.labels[item]"></el-input>
                </el-col>
                <el-col :span="6">
                  <i class="el-icon-delete" style="background: red" @click="removeKV(item)"/>
                </el-col>
              </el-row>
            </div>
            <div>
              <div class="kvgroup" v-if="showKV">
                <el-row :gutter="3">
                  <el-col :span="6" :gutter="3">
                <el-input v-model="newKey"></el-input>
                  </el-col>
                  <el-col :span="6" :gutter="3">
                <el-input v-model="newValue"></el-input>
                  </el-col>
                  <el-col :span="6" :gutter="3">
                    <i class="el-icon-delete" style="background: red" @click="clearKVCache"/>
                  </el-col>
                </el-row>
              </div>
              <el-button type="primary" class="buttonIcon" :icon="iconData"  @click="showKVbutton">{{buttonName}}</el-button>
            </div>
          </el-form-item>
          <el-divider>CONTAINER模板</el-divider>

          <div v-for="(container,i) in crdForm.spec.template.spec.containers" :key="i">
          <el-card shadow="hover">
          <el-form-item label="容器镜像"><el-input v-model="crdForm.spec.template.spec.containers[i].image"></el-input></el-form-item>
            <el-form-item label="容器名称"><el-input v-model="crdForm.spec.template.spec.containers[i].name"></el-input></el-form-item>
<!--           环境变量 -->

            <el-form-item label="容器环境">
              <div v-for="(envField,j) in container.env" :key="j">
                <el-row >
                  <el-col :span="12">
                  <el-input v-model="crdForm.spec.template.spec.containers[i].env[j].name"></el-input>
                  </el-col>
                  <el-col :span="12">
                  <el-input v-model="crdForm.spec.template.spec.containers[i].env[j].value"></el-input>
                  </el-col>
                </el-row>
<!--              <el-tag>{{crdForm.spec.template.spec.containers[i].env}}</el-tag>-->
<!--                <el-input> {{envList.value}}</el-input>-->
              </div>

            </el-form-item>

          </el-card>
          </div>
        </el-form>
        <div class="footer">
          <el-button type="primary" @click="confirmCrd">确定</el-button>
        </div>
      </el-tab-pane>
      <el-tab-pane>
        <span slot="label"><i class="el-icon-chat-line-square"/>YAML编辑</span>
        <el-button @click="printlog">测试</el-button>
        <div class="editor-container" style="height:700px;overflow:scroll">
          <yaml-editor v-model="value" ref="yamlEdit" :readOnly="readOnly"/>
        </div>
        <div class="footer">
          <el-button type="primary" @click="updateCrdByYaml">确定</el-button>
          <el-button type="primary">取消</el-button>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import YamlEditor from './yamlEditor.vue';
var json2yaml = require('json2yaml')
var jsyaml = require('js-yaml')
import {updateCrd} from "@/api/paasOperation";
export default {
  name: "crdEditor",
  props: ["crdForm","appgroupName","namespace","name"],
  components: {
    YamlEditor
  },
  data(){
    return {
      value: {},
      readOnly: false,
      newKey: "",
      newValue:"",
      showKV: false,
      iconData: "el-icon-document-add",
      iconColor: "blue",
      buttonName: "新增",
    }
  },
  methods:{
    printlog(){
      console.log("crdDorm",this.crdForm)
      this.value=json2yaml.stringify(this.crdForm)
    },
    async updateCrdByYaml(){
      let yam = JSON.parse(
          JSON.stringify(jsyaml.load(this.value), null, 4)
      );
      const res=await updateCrd({
        yaml: yam,
        clusterName: this.$parent.$parent.selectGroupName,
        namespace: this.$parent.$parent.namespace,
        name: "demo-asts"
      })
      if (res.code==0){
        this.$message.success("更新成功")
      }else{
        this.$message.warning(res.msg)
      }
    },
    replaceCrd(){

    },
    removeKV(item){
      // delete this.crdForm.spec.template.metadata.labels[item];
      this.$delete(this.crdForm.spec.template.metadata.labels,item)
      console.log("键值对",this.crdForm.spec.template.metadata.labels)
    },
    clearKVCache(){
      this.newValue="";
      this.newKey="";
      this.showKV=false;
      this.iconData="el-icon-document-add";
      this.buttonName="新增";
    },
    addKV(key,value){
      this.crdForm.spec.template.metadata.labels.push(key,value);
    },
    showKVbutton(){
      //如果是保存
      if (this.buttonName=="保存"){
        this.crdForm.spec.template.metadata.labels[this.newKey]=this.newValue;
        this.newKey="";
        this.newValue="";
        console.log("crd变化后：",this.crdForm.spec.template.metadata.labels)
      }
      this.showKV=!this.showKV;
      this.iconData=(this.iconData==="el-icon-refresh-left"?"el-icon-document-add":"el-icon-refresh-left");
      this.buttonName=(this.buttonName==="保存"?"新增":"保存");
      },
    getKey(obj){
      console.log("obj",Object.keys(obj))
    return Object.keys(obj);
    },
    getValue(obj){
      return Object.values(obj);
    },
    async confirmCrd(){
      const res=await updateCrd({
        yaml: this.crdForm,
        clusterName: this.$parent.$parent.selectGroupName,
        namespace: this.$parent.$parent.namespace,
        name: "demo-asts"
      })
      if (res.code==0){
        console.log("res",res)
      }
    },
    trigEdit(tab,event){
      if (tab.index==0){
        // this.$refs.yamlEdit.updateYaml();
        // this.crdForm=this.yamlData;
      }else if (tab.index==1){
        this.$nextTick(()=>{
          // this.value=json2yaml.stringify(this.crdForm) //用于yaml数据提交
          this.value = this.crdForm ? jsyaml.dump(this.crdForm) : "";
        })
      }
    },
    // yamlUpdate(val){
    //   console.log("修改值",val)
    //   this.yamlData=val
    // },
    toJson(val){
      console.log("val传参",val)
      console.log("crdForm",this.crdForm)
      let vm=this
      this.$set(vm,'yamlData',val)
      // this.yamlData=this.$refs.yamlEdit.getValue()
      let yaml = this.yamlData
      // console.log("yamldata:",this.$refs.yamlEdit.getValue())
      if (yaml) {
        try {
          let json = jsyaml.load(yaml)
          console.log("json:",json)
          this.$emit("updateCrdForm",json)
          this.$set(vm,'crdForm',json)
          console.log("vm擦书:",vm)
          // this.crdForm=json;
    } catch (e) {
      alert(e)
    }
  }
},
    show(val){
      console.log("输出输出",val)
    }
  }

}
</script>

<style scoped>

</style>
