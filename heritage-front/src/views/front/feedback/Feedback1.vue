<template>
  <div>
    <el-card style="margin-bottom:10px">
      请留言：
      <el-input v-model="data.commentContent" style="width: 400px"/>
      <el-button style="margin-left: 20px" type="danger" @click="submit">提交留言</el-button>
    </el-card>
    <el-card style="margin-bottom:10px">
      <el-table :data="data.tableData">
        <el-table-column label="头像">
          <template #default="scope">
            <img v-if="scope.row.user.avatar" :src="getAvatar(scope.row.user.avatar)" alt=""
                 style="display: block; width: 40px; height: 40px; border-radius: 50%"/>
          </template>
        </el-table-column>
        <el-table-column label="用户名" prop="user.username"/>
        <el-table-column label="留言内容" prop="commentContent"/>
        <el-table-column label="留言时间" prop="commentTime"/>
      </el-table>
      <div style="margin-top: 15px">
        <el-pagination
            @size-change="load"
            @current-change="load"
            v-model:current-page="data.pageNum"
            v-model:page-size="data.pageSize"
            :page-sizes="[5, 10, 15, 20]"
            layout="total, sizes, prev, pager, next, jumper"
            :total="data.total"/>
      </div>
    </el-card>
  </div>
</template>

<script setup>
import {reactive} from "vue";
import request from "@/utils/request.js";
import {ElMessage} from "element-plus";

const data = reactive({
  commentContent: '',
  tableData: [],
  pageNum: 1,
  pageSize: 10,
  total: 0,
  user: JSON.parse(localStorage.getItem("heritage-user"))
})

const getAvatar = (avatar) => {
  return request.defaults.baseURL + "/files/download/" + avatar;
}

const load = () => {
  request.get('/system/comment/selectPage', {
    params: {
      pageNum: data.pageNum,
      pageSize: data.pageSize,
      tag: '1'
    }
  }).then(res => {
    data.tableData = res.data.list;
    data.total = res.data.total
  })
}
load()

const submit = () => {
  if (data.commentContent === '') {
    ElMessage.error("留言不能为空")
    return
  }
  const now = new Date();
  const beijingTime = new Date(now.getTime() + 8 * 60 * 60 * 1000);
  const formattedDate = beijingTime.toISOString().slice(0, 19);

  request.post('/system/comment', {
    commentContent: data.commentContent,
    userId: data.user.id,
    tag: '0',
    commentTime: formattedDate
  }).then(res => {
    if (res.code === 200) {
      ElMessage.success('操作成功')
      load()
    } else {
      ElMessage.error(res.msg)
    }
  })
}
</script>