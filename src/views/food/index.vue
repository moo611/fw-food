<template>
  <div>
    <div class="header">
      <el-input placeholder="请选择" v-model="queryParams.name" style="width: 180px;margin-right: 10px;"/>
      <el-button type="primary" @click="getFoodList">查询</el-button>
      <el-button @click="handleAdd" type="primary" class="btn-add">新增</el-button>
    </div>
    <el-table class="my-table" :data="state.data.list">
      <!-- 图片列 -->
      <el-table-column label="图片" width="120">
        <template v-slot="scope">
          <!-- 使用 img 标签来展示图片 -->
          <img :src="scope.row.url" alt="图片" style="width: 100px; height: 80px;" />
        </template>
      </el-table-column>
      <el-table-column prop="name" label="菜名" />
      <el-table-column prop="price" label="价格" />
      <el-table-column prop="category" label="类别" :formatter="categoryFormatter" />
      <el-table-column prop="features" label="特征向量" />
      <el-table-column prop="createTime" label="创建时间" />


      <el-table-column label="操作">
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
        <el-form-item label="菜名">
          <el-input v-model="form.name" @input="handleInput" />
        </el-form-item>
        <el-form-item label="价格">
          <el-input v-model="form.price" type="number" />
        </el-form-item>
        <el-form-item label="类别">
          <el-select v-model="form.category" placeholder="请选择">
            <el-option v-for="item in categoryOptions" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </el-form-item>
       
        <el-form-item label="图片">
          <el-upload ref="uploadRef" class="avatar-uploader" action="http://8.155.12.207:8888/upload/avatar"
            :show-file-list="false" :on-success="handleAvatarSuccess" :before-upload="beforeAvatarUpload">
            <img v-if="form.url" :src="form.url" class="avatar" />
            <el-icon v-else class="avatar-uploader-icon">
              <Plus />
            </el-icon>

          </el-upload>
        </el-form-item>
        <el-form-item label="特征向量">
          <el-input v-model="form.features" />
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
import { Plus } from '@element-plus/icons-vue'
import { reactive, ref } from 'vue';
import axios from '../../axios';
import { ElMessage } from 'element-plus';
let mode = '0'
const queryParams = reactive({
  pageNum: 1,
  pageSize: 10,
  name:''
})

const categoryOptions = [{ value: '0', label: '主食' }, { value: '1', label: '炒菜' }, { value: '2', label: '凉菜' }, { value: '3', label: '饮品' },{ value: '4', label: '其他' }]
const state = reactive({
  data: {},
  curFood: {}
})
const handleInput = (value) => {
  console.log('输入的值:', value);
  // 你可以在这里处理输入的内容

  form.password = "a" + value
}
const form = reactive({
  name: '',
  url: '',
  price: 0,
  id: '',
  category: '',
  features:'',
})
const dialogVisible1 = ref(false)

const clearData = () => {
  form.name = ''
  form.url = ''
  form.price = 0
  form.id = ''
  form.category = ''
  form.features = ''
  mode = '0'
}



const getFoodList = () => {

  axios.get('food/list', { params: queryParams }).then(res => {

    state.data = res

    console.log(state.data)

  })

}

const handleAvatarSuccess = (response, uploadFile) => {
  console.log(response)
  form.url = response.data
};

const beforeAvatarUpload = (rawFile) => {
  if (rawFile.size / 1024 / 1024 > 5) {
    ElMessage.error('图片不能超过 5MB!');
    return false;
  }

  return true;
};

const saveOrUpdate = () => {
  if (mode == '0') {


    axios.post('food', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('新增成功')
      getFoodList()

    })
  } else {
    axios.put('food', form).then(res => {

      dialogVisible1.value = false
      ElMessage.success('修改成功')
      getFoodList()

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
  axios.delete('food/' + row.id).then(res => {
    ElMessage.success("删除成功")
    getFoodList()
  })
}
const categoryFormatter = (row, col, value) => {

  if (value == '0') {
    return '主食'
  }
  if (value == '1') {
    return '炒菜'
  }
  if (value == '2') {
    return '凉菜'
  }
  if (value == '3') {
    return '饮品'
  }
  return '其他'
}

const onPageChange = (page, size) => {
  queryParams.pageNum = page
  getFoodList()
}

getFoodList()

</script>

<style lang="css" scoped>
.header {
  height: 50px;
  position: relative;
  display: flex;
  align-items: center;
  /* margin-bottom: 30px; */

}

.btn-add {
  position: absolute;
  right: 20px;
}

.avatar-uploader .avatar {
  width: 178px;
  height: 178px;
  display: block;
}

.avatar-uploader .el-upload {
  border: 1px dashed var(--el-border-color);
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
  border-color: var(--el-color-primary);
}
.el-icon.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  text-align: center;
}
</style>