<template>
  <div>
    <div class="header">
      <span style="margin-right: 5px;">角色</span>  
      <el-select style="width: 150px;" v-model="queryParams.role" placeholder="请选择角色" >
        <el-option v-for="item in roleOptions" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
      <el-input v-model="queryParams.nickname" placeholder="请输入" style="width: 180px;margin-left: 10px;margin-right: 10px;"/>
      <el-button @click="getUserList" type="primary" >查询</el-button>
    </div>
    <el-table class="my-table" :data="state.data.list">

      <el-table-column prop="username" label="用户名" />
      <el-table-column prop="nickname" label="昵称" />
      <el-table-column prop="createTime" label="创建时间" />
      <el-table-column prop="role" label="角色" width="80" :formatter="roleFormatter" />
      <el-table-column label="操作" width="180">
        <template #default="scope">
          <el-button type="primary" @click="handleEdit(scope.$index, scope.row)">
            编辑
          </el-button>
          <el-button type="danger" @click="handleDel(scope.$index, scope.row)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination layout="prev, pager, next" :total="state.data.total" :page-size="queryParams.pageSize"
      @change="onPageChange" />

    <el-dialog v-model="dialogVisible1" width="500" @close="clearData">

      <el-form class="form" :model="form" label-width="auto" style="max-width: 600px">
        <el-form-item label="用户名">
          <el-input v-model="form.username" :disabled="mode == '1'" />
        </el-form-item>
        <!-- <el-form-item label="密码">
          <el-input v-model="form.password" />
        </el-form-item> -->
        <el-form-item label="昵称">
          <el-input v-model="form.nickname" />
        </el-form-item>

        <el-form-item label="角色">
          <el-select v-model="form.role" placeholder="请选择角色">
            <el-option v-for="item in roleOptions" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </el-form-item>
       

      </el-form>

      <template #footer>
        <div class="dialog-footer">
          <el-button @click="dialogVisible1 = false">取消</el-button>
          <el-button type="primary" @click="saveOrUpdate">
            确定
          </el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script lang="js" setup>
import { reactive, ref } from 'vue';

import axios from '../../axios';
import { ElMessage } from 'element-plus';
let mode = '0'
const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,
  role: 'all',
  nickname:''
})
const state = reactive({
  data: {},
  curUser: {}
})

const form = reactive({
  username: '',
  password: '',
  nickname: '',
  role: '0',
 
  id: ''

})
const dialogVisible1 = ref(false)

const clearData = () => {
  form.username = ''
  form.password = ''
  form.nickname = ''
  form.role = '0'
  
  form.id = ''
  mode = '0'
}
const roleOptions = [
  {value: 'all',
  label: '全部',},
  {
    value: '0',
    label: '管理员',
  },
  {
    value: '1',
    label: '学生',
  },]



const roleFormatter = (row, column, cellValue) => {
  if (cellValue == '0') {
    return '管理员'
  } else if (cellValue == '1') {
    return '学生'
  }
  return ''
}


const getUserList = () => {
  let params = {...queryParams}

  if(queryParams.role == 'all'){
    params.role = null
  }
  axios.get('user/list', { params: params }).then(res => {

    state.data = res

    console.log(state.data)

  })

}

const saveOrUpdate = () => {
  if (mode == '0') {

    if (form.role == '') {
      ElMessage.error('请选择角色')
      return
    }


    axios.post('user', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('新增成功')
      getUserList()


    })
  } else {
    axios.put('user', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('修改成功')
      getUserList()


    })
  }


}

const copyValue = (src, target) => {
  // 遍历 target 中的 key，并将 src 对应属性赋值给 target
  Object.keys(target).forEach((key) => {
    if (src[key] !== undefined) {
      target[key] = src[key] // 仅赋值存在于 src 中的属性
    }
  })
}

const handleAdd = () => {
  mode = '0'
  dialogVisible1.value = true
}

const handleEdit = (index, row) => {
  mode = '1'
  copyValue(row, form)
  dialogVisible1.value = true
}
const handleDel = (index, row) => {
  axios.delete('user/' + row.id).then(res => {
    ElMessage.success("删除成功")
    getUserList()
  })
}

const onPageChange = (page, size) => {
  queryParams.pageNum = page
  getUserList()
}

getUserList()

</script>

<style lang="css" scoped>
.header {
  height: 50px;
  position: relative;
  display: flex;
  align-items: center;
  margin-bottom: 30px;

}

.btn-add {
  position: absolute;
  right: 20px;
}
</style>