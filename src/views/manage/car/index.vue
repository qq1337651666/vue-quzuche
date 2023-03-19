<template>
  <div class="app-container">

    <el-row :gutter="20" class="header">
      <el-col :span="7">
        <el-input placeholder="请输入车牌号码..." v-model="queryForm.query" clearable></el-input>
      </el-col>
      <el-button type="primary" :icon="Search" @click="initCarList">搜索</el-button>
      <el-button type="success" :icon="DocumentAdd" @click="handleDialogValue()" v-if="hasAuth('system:car:add')">新增
      </el-button>

    </el-row>
    <el-table
        :data="tableData"
        stripe
        style="width: 100%"
        @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55"/>
      <el-table-column prop="carImg" label="汽车图" width="80" align="center">
        <template v-slot="scope">
          <img :src="'https://hard-dog.oss-cn-shenzhen.aliyuncs.com/'+ scope.row.carImg" width="50" height="50"/>
        </template>
      </el-table-column>
      <el-table-column prop="carNumber" label="车牌号" width="100" align="center"/>
      <el-table-column prop="carType" label="车类型" width="70" align="center">
        <template v-slot="scope">
          <el-tag size="small" type="warning"> {{ carTypesMap[scope.row.carType] }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="carColor" label="车颜色" width="70" align="center"/>
      <el-table-column prop="carPrice" label="价格" width="80" align="center"/>
      <el-table-column label="租借状态？" width="200" align="center">
        <template v-slot="{row}">
          <el-switch v-model="row.isRenting" @change="statusChangeHandle(row)" active-text="已租"
                     inactive-text="可租" :active-value="0" :inactive-value="1"></el-switch>
        </template>
      </el-table-column>
      <el-table-column prop="createTime" label="入库时间" width="200" align="center">
        <template v-slot="{row}">
          <span>{{ new Date(row.createTime).toLocaleDateString().replaceAll("/", "-") }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="description" label="描述"/>
      <el-table-column prop="action" label="操作" width="200" fixed="right" align="center">
        <template v-slot="scope">
          <!--          <el-button  type="primary" :icon="Tools" @click="handleRoleDialogValue(scope.row.id,scope.row.sysRoleList)" v-if="hasAuth('system:user:role')">分配角色</el-button>-->

          <!--          <el-popconfirm  v-if="scope.row.username!='java1234'" title="您确定要对这个用户重置密码吗？" @confirm="handleResetPassword(scope.row.id)">-->
          <!--            <template #reference>-->
          <!--              <el-button  type="warning" :icon="RefreshRight" v-if="hasAuth('system:user:resetPwd')">重置密码</el-button>-->
          <!--            </template>-->
          <!--          </el-popconfirm>-->

          <el-button v-if="hasAuth('system:car:edit')" type="primary" :icon="Edit"
                     @click="handleDialogValue(scope.row.id)"/>

          <el-popconfirm v-if="scope.row.username!='java1234'" title="您确定要删除这条记录吗？"
                         @confirm="handleDelete(scope.row.id)">
            <template #reference>
              <el-button type="danger" :icon="Delete" v-if="hasAuth('system:user:delete')"/>
            </template>
          </el-popconfirm>


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
  <Dialog v-model="dialogVisible" :dialogVisible="dialogVisible" :id="id" :carTypes="carTypes"
          :dialogTitle="dialogTitle" @initCarList="initCarList"></Dialog>

  <!--  <Dialog v-model="dialogVisible" :dialogVisible="dialogVisible" :id="id" :dialogTitle="dialogTitle" @initCarList="initCarList"></Dialog>-->
  <!--  <RoleDialog v-model="roleDialogVisible" :sysRoleList="sysRoleList" :roleDialogVisible="roleDialogVisible" :id="id" @initCarList="initCarList"></RoleDialog>-->

</template>

<script setup>
import {Search, Delete, DocumentAdd, Edit, Tools, RefreshRight} from '@element-plus/icons-vue'
import {ref} from 'vue'
import requestUtil, {getServerUrl} from "@/utils/request";
import {ElMessage, ElMessageBox} from 'element-plus'
import Dialog from "@/views/manage/car/components/dialog.vue";


const queryForm = ref({
  query: '',
  pageNum: 1,
  pageSize: 10
})

const total = ref(0)

const tableData = ref([])

const id = ref(-1)

const sysRoleList = ref([])

const imageDialogValue = ref({})

const dialogVisible = ref(false)

const roleDialogVisible = ref(false)

const dialogTitle = ref('')

const multipleSelection = ref([])

const delBtnStatus = ref(true)

const carTypes = ref([]);
const carTypesMap = ref([]);

const handleSelectionChange = (selection) => {
  console.log("勾选了")
  console.log(selection)
  multipleSelection.value = selection;
  delBtnStatus.value = selection.length == 0;
}



const initCarList = async () => {
  const res = await requestUtil.post("car/list", queryForm.value)
  tableData.value = res.data.list;
  console.log(res.data.list)
  const types = {} // map
  // 转map  == js object
  // key:id value:类型名
  // {1:"轿车",2:"SUV“}
  // cartype:2
  for (let type of res.data.carTypes) {
    types[type.id] = type.typeName;
  }
  carTypesMap.value = types;
  carTypes.value = res.data.carTypes;
  total.value = res.data.total;
}

//执行初始化方法 赋值
initCarList();

const handleSizeChange = (pageSize) => {
  queryForm.value.pageNum = 1;
  queryForm.value.pageSize = pageSize;
  initCarList();
}

const handleCurrentChange = (pageNum) => {
  queryForm.value.pageNum = pageNum;
  initCarList();
}


const statusChangeHandle = async (row) => {
  let res = await requestUtil.get("/car/update/" + row.id + "/status/" + row.isRenting);
  if (res.data.code == 200) {
    ElMessage({
      type: 'success',
      message: '执行成功!'
    })
  } else {
    ElMessage({
      type: 'error',
      message: res.data.msg,
    })
    initCarList();
  }
}

const handleDialogValue = (carId) => {
  if (carId) {
    id.value = carId;
    dialogTitle.value = "修改车辆"
  } else {
    id.value = -1;
    dialogTitle.value = "添加车辆"
  }
  dialogVisible.value = true
}

const handleRoleDialogValue = (userId, roleList) => {
  console.log("userId=" + userId)
  id.value = userId;
  sysRoleList.value = roleList;
  roleDialogVisible.value = true
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
  const res = await requestUtil.post("car/delete", ids)
  if (res.data.code == 200) {
    ElMessage({
      type: 'success',
      message: '执行成功!'
    })
    initCarList();
  } else {
    ElMessage({
      type: 'error',
      message: res.data.msg,
    })
  }
}

const handleResetPassword = async (id) => {
  const res = await requestUtil.get("sys/user/resetPassword/" + id)
  if (res.data.code == 200) {
    ElMessage({
      type: 'success',
      message: '执行成功!'
    })
    initCarList();
  } else {
    dialogVisible
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
