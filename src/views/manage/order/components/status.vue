<template>
  <div>
    <el-dialog v-model="statusDialogVisible" :title="statusTitle" width="30%" draggable @close="handleClose">
      <!--      <span>It's a draggable Dialog</span>-->
      <!--      <template #footer>-->
      <!--      </template>-->
      <el-form
          ref="formRef"
          :model="orderForm"
          :rules="rules"
          label-width="100px"
      >
        <el-form-item label="用户名称" prop="username">
          <el-input v-model="orderForm.username" disabled/>
        </el-form-item>

        <el-form-item label="联系方式" prop="phonenumber">
          <el-input v-model="orderForm.phonenumber" disabled/>
        </el-form-item>

        <el-form-item label="订单状态" prop="orderStatus">
          <el-select v-model="orderForm.orderStatus">
            <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
                :disabled="item.disabled"
            />
          </el-select>
        </el-form-item>

        <el-form-item label="租赁时间">
          <div class="demo-date-picker">
            <div class="block">
              <!--              {{ orderForm.startDate }}-->
              <el-date-picker
                  v-model="datePick"
                  type="daterange"
                  range-separator="To"
                  start-placeholder="Start date"
                  end-placeholder="End date"
                  :default-value="datePick"
                  :value-format="YYYY-MM-DD"
                  @change="datePickHandler"
              />
            </div>
          </div>
        </el-form-item>

      </el-form>
      <span class="dialog-footer">
                <el-button @click="statusDialogVisible = false">Cancel</el-button>
                <el-button type="primary" @click="confirmHandler">
                  Confirm
                </el-button>
              </span>
    </el-dialog>
  </div>
</template>

<script setup>

import {defineEmits, ref, watch} from "vue";
import requestUtil, {getServerUrl} from "@/utils/request";
import {ElMessage} from 'element-plus'

const size = ref < 'default' | 'large' | 'small' > ('default')

const value1 = ref('')
const value2 = ref('')
const formRef = ref(null)
const options = [
  {
    value: 0,
    label: '审核不通过~~',
  },
  {
    value: 1,
    label: '租借中~~',
  },
  {
    value: 2,
    label: '已完成~~',
  },
  {
    value: 3,
    label: '审核中~~',
  },
]


const orderForm = ref({
  id: -1,
  username: "",
  phonenumber: "",
  startDate: "",
  endDate: "",
  orderStatus: ""
})

const confirmHandler = () => {
  handleConfirm();
  props.statusDialogVisible = false;
  // console.log(`日期：${orderForm.value.startDate} - ${orderForm.value.endDate}`)
}

const datePickHandler = () => {
  orderForm.value.startDate = datePick.value[0];
  orderForm.value.endDate = datePick.value[1];

}

const datePick = ref();


const emits = defineEmits(['update:modelValue', 'initOrderList'])

const handleClose = () => {
  emits('update:statusDialogVisible', false)
}


watch(
    () => props.statusDialogVisible,
    () => {
      console.log(props.statusDialogVisible);
      if (props.row != null) {
        console.log(datePick.value)
        orderForm.value = props.row;
        datePick.value = [new Date(orderForm.value.startDate), new Date(orderForm.value.endDate)];

      } else {

        form.value = {
          id: -1,
          username: "",
          phonenumber: "",
          startDate: "",
          endDate: "",
          orderStatus: ""
        }

      }
    }
)

const props = defineProps(
    {
      id: {
        type: Number,
        default: -1,
        required: true
      },
      statusDialogVisible: {
        type: Boolean,
        default: false,
        required: true
      },
      statusTitle: {
        type: String,
        default: "弹框",
        required: true
      },
      row: {},
    }
)

const handleConfirm = () => {
  formRef.value.validate(async (valid) => {
    if (valid) {
      let result = await requestUtil.post("/manage/order/updateOrder", orderForm.value);
      let data = result.data;
      if (data.code == 200) {
        ElMessage.success("执行成功")
        formRef.value.resetFields();
        emits("initOrderList")
        handleClose();
      } else {
        ElMessage.error(data.msg)
      }
    }
  })
}

</script>

<style scoped>
.demo-date-picker {
  display: flex;
  width: 100%;
  padding: 0;
  flex-wrap: wrap;
}

.demo-date-picker .block {
  padding: 30px 0;
  text-align: center;
  border-right: solid 1px var(--el-border-color);
  flex: 1;
}

.demo-date-picker .block:last-child {
  border-right: none;
}

.demo-date-picker .demonstration {
  display: block;
  color: var(--el-text-color-secondary);
  font-size: 14px;
  margin-bottom: 20px;
}
</style>
