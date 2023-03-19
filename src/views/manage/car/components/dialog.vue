<template>
  <el-dialog
    model-value="dialogVisible"
    :title="dialogTitle"
    width="30%"
  @close="handleClose"
  >

    <el-form
        ref="formRef"
        :model="form"
        :rules="rules"
        label-width="100px"
    >
    <el-form-item label="汽车照">
      <el-upload
          :headers="headers"
          class="avatar-uploader"
          :action="getServerUrl()+'car/uploadImageToOSS'"
          :show-file-list="false"
          :on-success="handleAvatarSuccess"
          v-model="form.carImg"
      >
        <img v-if=" form.carImg" :src="'https://hard-dog.oss-cn-shenzhen.aliyuncs.com/' +form.carImg + 't=' + new Date().getTime()" class="avatar" />
        <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
      </el-upload>
    </el-form-item>
      <el-form-item label="车牌号" prop="carNumber">
        <el-input v-model="form.carNumber" />
      </el-form-item>

      <el-form-item label="类型" prop="carType">
        <el-select v-model="form.carType">
          <template v-for="carType in carTypes">
            <el-option :label="carType.typeName" :value="carType.id"></el-option>
          </template>
        </el-select>
      </el-form-item>

      <el-form-item label="颜色" prop="carColor">
        <el-input v-model="form.carColor" />
      </el-form-item>

      <el-form-item label="汽车价格" prop="carPrice">
        <el-input v-model="form.carPrice" />
      </el-form-item>

      <el-form-item label="租赁价格" prop="rentPrice">
        <el-input v-model="form.rentPrice" />
      </el-form-item>

      <el-form-item label="描述" prop="description">
        <el-input v-model="form.description" type="textarea" :rows="4"/>
      </el-form-item>


    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button type="primary" @click="handleConfirm">确认</el-button>
        <el-button  @click="handleClose">取消</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup>

import {defineEmits, defineProps, reactive, ref, watch} from "vue";
import requestUtil,{getServerUrl} from "@/utils/request";
import { ElMessage } from 'element-plus'
import store from "@/store";
import { Plus } from '@element-plus/icons-vue'

  const tableData=ref([])

const headers=ref({
  token:store.getters.GET_TOKEN
})


  const props=defineProps(
      {
        id:{
          type:Number,
          default:-1,
          required:true
        },
        dialogTitle:{
          type:String,
          default:'',
          required:true
        },
        dialogVisible:{
          type:Boolean,
          default:false,
          required:true
        },
        carTypes:{

        }
      }
  )


const form=ref({
  id:-1,
  carNumber:"",
  carType:1,
  carColor:"",
  carPrice:"",
  rentPrice:"",
  remark:"",
  carImg:"",
  description:"",
  isRenting:0
})
const handleAvatarSuccess=(res)=>{
  form.value.carImg=res.data.src;
  console.log(res.data.src)
}


// const checkUsername = async (rule, value, callback) => {
//   if(form.value.id==-1){
//     const res=await requestUtil.post("sys/user/checkUserName",{username:form.value.username});
//     if (res.data.code==500) {
//       callback(new Error("用户名已存在！"));
//     } else {
//       callback();
//     }
//   }else{
//     callback();
//   }
//
// }


const rules=ref({
  carNumber:[
    { required: true, message: '请输入车牌号'},
  ],
  carType: [{ required: true, message: "请输入汽车类型", trigger: "blur" }],
  carColor: [{ required: true, message: "请输入颜色", trigger: "blur" }],
  carPrice: [{ required: true, message: "请输入汽车价格", trigger: "blur" }],
})

const formRef=ref(null)

const carType = ref([]);

const initFormData=async(id)=>{
  const res=await requestUtil.get("car/"+id);
  form.value=res.data.car;
  carType.value = res.data.carTypes;
}
watch(
    ()=>props.dialogVisible,
    ()=>{
      let id=props.id;
      if(id!=-1){
        initFormData(id)
      }else{
        console.log("change")
        form.value={
          id:-1,
          carNumber:"",
          carType:1,
          carColor:"",
          carPrice:"",
          rentPrice:"",
          remark:"",
          carImg:"",
          description:"",
          isRenting:0
        }

      }
    }
)






  const emits=defineEmits(['update:modelValue','initUserList'])

  const handleClose=()=>{
    emits('update:modelValue',false)
  }

  const handleConfirm=()=>{
    formRef.value.validate(async(valid)=>{
      console.log(form.value)
      if(valid){
          let result=await requestUtil.post("car/add",form.value);
          let data=result.data;
          if(data.code==200){
            ElMessage.success("执行成功！")
            formRef.value.resetFields();
            emits("initCarList")
            handleClose();
          }else{
            ElMessage.error(data.msg);
          }
      }else{
        console.log("fail")
      }
    })
  }

</script>

<style scoped>

.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.el-icon.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  text-align: center;
}
.avatar {
  width: 120px;
  height: 120px;
  display: block;
}

</style>
