<template>
  <div style="width: 70%;margin-left: 10px;margin-top: 10px;display: flex;">
    <el-image style="width: 300px; height: 400px;margin-right:20px" :src="'https://hard-dog.oss-cn-shenzhen.aliyuncs.com/'+ state.car.carImg" :fit="fit" />
    <div>
      <div style="display: flex;width:300px;margin-bottom: 20px; font-size: 20px; justify-content: space-between"><span>车型号:</span><span>{{state.car.description}}</span></div>
      <div style="display: flex;width:300px;margin-bottom: 20px;font-size: 20px; justify-content: space-between"><span>原价:</span><span>{{state.car.carPrice}}</span></div>
      <div style="display: flex;width:300px;margin-bottom: 20px;font-size: 20px; justify-content: space-between"><span>租车更优惠:</span><span>{{state.car.rentPrice}}</span></div>
      <div style="display: flex;width:300px;margin-bottom: 20px;font-size: 20px; justify-content: space-between"><span>汽车颜色:</span><span>{{state.car.carColor}}</span></div>
      <el-date-picker
          v-model="state.datePick"
          type="daterange"
          range-separator="To"
          start-placeholder="Start date"
          end-placeholder="End date"
          :default-value="datePick"
          :value-format="YYYY-MM-DD"
          @change="datePickHandler"
      />

      <div style="width: 320px;display: flex;justify-content: right">
        <el-button type="error" style="margin-top:20px">下单</el-button>
      </div>
    </div>
  </div>
</template>


<script setup>

import {useRoute} from "vue-router";
import requestUtil from "@/utils/request";
import {onMounted, reactive} from "vue";

const route = useRoute();

const id = route.query.id;

const state = reactive({
  carTypeMap:{},
  car:[],
  carType:[],
  datePick:[]
})
const initList = async () => {
  const res = await requestUtil.get("/car/" + id, {})
  state.car = res.data.car
  state.carType = res.data.carTypes

  res.data.carTypes.forEach(item=>{
    state.carTypeMap[item.id] = item.typeName;
  })

}

onMounted(()=>{
  initList();
})

const getServerUrl = ()=>{
  return "http://localhost:80/";
}


</script>
