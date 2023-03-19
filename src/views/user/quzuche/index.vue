<template>
  <div class="app-container layout">
   <div class="header">
     <el-form-item label="汽车类型">
       <el-select v-model="state.carTypeSearch" class="m-2" placeholder="Select">
         <el-option
             v-for="item in state.carype"
             :key="item.value"
             :label="item.typeName"
             :value="item.id"
         />
       </el-select>
     </el-form-item>
   </div>

       <div class="car-list">
         <div v-for="item in state.list" >
           <el-card :body-style="{ padding: '0px' }" class="item">
             <img
                 :src="'https://hard-dog.oss-cn-shenzhen.aliyuncs.com/' +item.carImg"
                 class="image"
                 style="height: 300px"
             />
             <div style="padding: 14px">
               <span style="font-weight: bold;font-size: 25px;display: flex;align-items: center">{{ item.description }}
               <el-tag type="success" style="margin-left:10px">{{state.carTypeMap[item.carType]}}</el-tag>
               </span>
               <div style="margin-top: 20px;font-size: 20px;color:#ff5000;display: flex;justify-content: space-between">
                 <span>￥{{item.rentPrice}}</span>
                 <el-button class="button" style="padding:0 10px" type="info" @click="router.push({path:'/car/detail',query:{id:item.id}})" >详情</el-button></div>

             </div>
           </el-card>
         </div>
       </div>
  </div>
</template>

<script setup>
import requestUtil from "@/utils/request";
import {onMounted, reactive} from "vue";
import {useRouter} from "vue-router";

const router = useRouter();
const getServerUrl = ()=>{
  return "http://localhost:80/";
}

const state = reactive({
  list:[],
  carType:[],
  carTypeMap:{}
})


onMounted(()=>{
  initList();
})

const initList = async () => {
  const res = await requestUtil.post("/car/list", {})
  state.list = res.data.list
  state.carype = res.data.carTypes

  res.data.carTypes.forEach(item=>{
    state.carTypeMap[item.id] = item.typeName;
  })

}

</script>

<style lang="scss" scoped>
.time {
  font-size: 12px;
  color: #999;
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.button {
  padding: 0;
  min-height: auto;
}

.image {
  width: 100%;
  display: block;
}

.item{
  width: 350px;
  height: 400px;
}

.car-list{
  display: grid;
  grid-template-columns: repeat(4, 400px );
  grid-template-rows:  repeat(4, 500px);
}

</style>
