<template>
  <div>
    <div>
      <el-card shadow="never" style="margin-bottom: 10px">
        <el-table :data="data.tableData">
          <el-table-column label="ID" prop="commentId"/>
          <el-table-column label="头像">
            <template #default="scope">
              <img v-if="scope.row.user.avatar" :src="getAvatar(scope.row.user.avatar)" alt=""
                   style="display: block; width: 40px; height: 40px; border-radius: 50%"/>
            </template>
          </el-table-column>
          <el-table-column label="用户名" prop="user.username"/>
          <el-table-column label="留言内容" prop="commentContent"/>
          <el-table-column label="留言时间" prop="commentTime"/>
          <el-table-column label="审核上架">
            <template #default="scope">
              <el-switch v-model="scope.row.tag" :active-value="1" :inactive-value="0"
                         @change="update(scope.row.tag, scope.row.commentId)"
                         inline-prompt active-text="通过" inactive-text="不通过"/>
            </template>
          </el-table-column>
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
  </div>
</template>

<script setup>
import {reactive} from "vue";
import request from "@/utils/request.js";
import {ElMessage} from "element-plus";

const data = reactive({
  commentId: null,
  tag: null,
  tableData: [],
  pageNum: 1,
  pageSize: 10,
  total: 0,
})

const getAvatar = (avatar) => {
  return request.defaults.baseURL + "/files/download/" + avatar;
}

const load = () => {
  request.get('/system/comment/selectPage', {
    params: {
      pageNum: data.pageNum,
      pageSize: data.pageSize,
    }
  }).then(res => {
    data.tableData = res.data.list
    data.total = res.data.total
  })
}
load()

const update = (tagg, idd) => {
  request.put('/system/comment', {
    commentId: idd,
    tag: tagg,
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