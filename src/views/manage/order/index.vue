<template>
  <div class="app-container">

    <el-row :gutter="20" class="header">
      <el-col :span="7">
        <el-form-item label="订单状态" prop="orderStatus">
          <el-select v-model="queryForm.query">
            <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
                :disabled="item.disabled"
            />
          </el-select>

        </el-form-item>
        <!--          <el-input placeholder="请输入角色名..." v-model="queryForm.query" clearable ></el-input>-->
      </el-col>

      <el-button type="primary" :icon="Search" @click="initRoleList">搜索</el-button>
    </el-row>
    <el-table
        :data="tableData"
        stripe
        style="width: 100%"
        @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55"/>
      <el-table-column prop="username" label="用户" width="100" align="center"/>
      <el-table-column prop="phonenumber" label="联系方式" width="200" align="center"/>
      <!--        <el-table-column prop="vehicleInformation" label="订单状态？" width="100" align="center" >-->
      <el-table-column prop="username" label="用户" width="100" align="center"/>
      <el-table-column label="订单状态" width="100" align="center">
        <template v-slot="{row}">
          <div v-show="row.orderStatus===1">租借中~~</div>
          <div v-show="row.orderStatus===2">租借完成~~</div>
          <div v-show="row.orderStatus===3">审核状态~~</div>
        </template>
      </el-table-column>
      <el-table-column prop="order" label="创建时间" width="100" align="center">
        <template v-slot="{row}">
          <span>{{ new Date(row.createDate).toLocaleDateString().replaceAll("/", "-") }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="startDate" label="租借时间" width="200" align="center">
        <template v-slot="{row}">
            <span>{{ new Date(row.startDate).toLocaleDateString().replaceAll("/", "-") }}&nbsp;至&nbsp;
            {{ new Date(row.endDate).toLocaleDateString().replaceAll("/", "-") }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="carNumber" label="车牌"/>
      <el-table-column prop="rentPrice" label="租金" width="100" fixed="right" align="center"/>
      <el-table-column prop="action" label="操作" width="200" fixed="right" align="center">
        <template v-slot="scope">
<!--          <el-button v-if="scope.row.code!='admin' && hasAuth('system:role:edit')" type="primary" :icon="Edit"-->
<!--                     @click="handleDialogValue(scope.row.id)"/>-->
          <el-button type="primary" :icon="Edit" @click="statusDialogHandler(scope.row)"/>
<!--          <el-popconfirm v-if="scope.row.code!='admin'" title="您确定要删除这条记录吗？"-->
<!--                         @confirm="handleDelete(scope.row.id)">-->
<!--            <template #reference>-->
<!--              <el-button type="danger" :icon="Delete" v-if="hasAuth('system:role:delete')"/>-->
<!--            </template>-->
<!--          </el-popconfirm>-->

        </template>
      </el-table-column>
    </el-table>
    <el-pagination
        v-model:currentPage="queryForm.pageNum"
        v-model:page-size="queryForm.pageSize"
        :page-sizes="[10, 20, 30, 40,50]"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
    />
  </div>
<!--  <Dialog v-model="dialogVisible" :dialogVisible="dialogVisible" :id="id" :dialogTitle="dialogTitle"-->
<!--          @initRoleList="initRoleList"></Dialog>-->
  <Status :row="state.currentRow" v-model:status-dialog-visible="statusDialogVisible" :statusTitle="statusTitle"></Status>

  <MenuDialog v-model="menuDialogVisible" :menuDialogVisible="menuDialogVisible" :id="id"
              @initRoleList="initRoleList"></MenuDialog>


</template>

<script setup>
import {Search, Delete, DocumentAdd, Edit, Tools, RefreshRight} from '@element-plus/icons-vue'
import {watch, reactive, ref} from 'vue'
import requestUtil, {getServerUrl} from "@/utils/request";
import {ElMessage, ElMessageBox} from 'element-plus'
import Dialog from './components/dialog'
import Status from './components/status'


const queryForm = ref({
  query: '',
  pageNum: 1,
  pageSize: 10
})

// const value = ref('')

const total = ref(0)

const tableData = ref([])

const id = ref(-1)

const sysRoleList = ref([])

const imageDialogValue = ref({})

const dialogVisible = ref(false)

const statusTitle = ref('')

const statusDialogVisible = ref(false)

const menuDialogVisible = ref(false)

const dialogTitle = ref('')

const multipleSelection = ref([])

const delBtnStatus = ref(true)

const options = [
  {
    value: '',
    label: "全部订单",
  },
  {
    value: '1',
    label: '租借中~~',
  },
  {
    value: '2',
    label: '已完成',
  },
  {
    value: '3',
    label: '审核状态',
  },
]

const handleSelectionChange = (selection) => {
  console.log("勾选了")
  console.log(selection)
  multipleSelection.value = selection;
  delBtnStatus.value = selection.length == 0;
}

const initRoleList = async () => {
  const res = await requestUtil.post("manage/order/listAll", queryForm.value)
  tableData.value = res.data.orderList;
  total.value = res.data.total;
}



initRoleList();

const handleSizeChange = (pageSize) => {
  queryForm.value.pageNum = 1;
  queryForm.value.pageSize = pageSize;
  initRoleList();
}

const handleCurrentChange = (pageNum) => {
  queryForm.value.pageNum = pageNum;
  initRoleList();
}

const handleDialogValue = (roleId) => {
  if (roleId) {
    id.value = roleId;
    dialogTitle.value = "角色修改"
  } else {
    id.value = -1;
    dialogTitle.value = "角色添加"
  }
  dialogVisible.value = true
}

const state = reactive({
  currentRow:{}
})
const statusDialogHandler = (row) => {
  statusTitle.value = "修改订单信息";
  state.currentRow = row;
  statusDialogVisible.value = true;

}

const handleMenuDialogValue = (roleId) => {
  if (roleId) {
    id.value = roleId;
  }
  menuDialogVisible.value = true
}

const handleDelete = async (id) => {
  var ids = []
  if (id) {
    ids.push(id)
  } else {
    multipleSelection.value.forEach(row => {
      ids.push(row.id)
    })
  }
  const res = await requestUtil.post("sys/role/delete", ids)
  if (res.data.code == 200) {
    ElMessage({
      type: 'success',
      message: '执行成功!'
    })
    initRoleList();
  } else {
    ElMessage({
      type: 'error',
      message: res.data.msg,
    })
  }
}


</script>

<style lang="scss" scoped>

.header {
  padding-bottom: 16px;
  box-sizing: border-box;
}

.el-pagination {
  float: right;
  padding: 20px;
  box-sizing: border-box;
}

::v-deep th.el-table__cell {
  word-break: break-word;
  background-color: #f8f8f9 !important;
  color: #515a6e;
  height: 40px;
  font-size: 13px;

}

.el-tag--small {
  margin-left: 5px;
}

</style>
