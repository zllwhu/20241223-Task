<template>
  <div style="display: flex; justify-content: center; padding-top: 30px; flex-direction: column">
    <el-space v-if="data.total" wrap style="display: flex; justify-content: center; margin-bottom: 30px" size="large">
      <el-card class="box-card" style="width: 260px" @click="loadWhu">
        <template #header>
          <div class="card-header">
            <span>武汉大学建筑</span>
          </div>
        </template>
        <div style="display: flex; justify-content: center">
          <img style="width: 240px; height: 180px" src="@/assets/img/武汉大学早期建筑分布图.svg" alt=""/>
        </div>
      </el-card>
      <el-card v-for="(item, index) in data.allData" :key="index" class="box-card" style="width: 260px">
        <template #header>
          <div class="card-header">
            <span>{{ item.archiName }}</span>
          </div>
        </template>
        <div style="display: flex; justify-content: center">
          <el-empty v-if="!item.archiCover" :image-size="55"/>
          <img v-if="item.archiCover" style="width: 240px; height: 180px" :src="getAvatar(item)" alt=""
               @click="handleImagePreview(item)"/>
        </div>
      </el-card>
    </el-space>
  </div>

  <el-dialog v-model="data.dialogVisible" width="700px" @close="data.currentImage = ''">
    <div class="image-container">
      <img :src="data.currentImage" alt="预览图" style="width: 668px; height: 501px;"/>
    </div>
  </el-dialog>

  <el-dialog v-model="data.dialogWhuVisible" width="79%" @close="data.currentImage = ''">
    <RouterView/>
  </el-dialog>
</template>

<script setup>
import {reactive} from "vue";
import request from "@/utils/request.js";
import router from "@/router/index.js";

const data = reactive({
  allData: [],
  total: 0,
  dialogVisible: false,
  dialogWhuVisible: false,
  currentImage: '',
})

// 加载数据并根据分页参数传递分页信息
const load = () => {
  request.get('/system/architecture/selectPage', {
    params: {
      pageNum: 1,
      pageSize: 100000,
    }
  }).then(res => {
    if (res.data) {
      data.allData = res.data.list;  // 存储全部数据
      data.total = res.data.total;  // 总数据条数
      console.log(res)
    } else {
      console.error("接口返回的数据格式不正确");
    }
  }).catch(error => {
    console.error("请求失败:", error);
  });
};
load();

const getAvatar = (item) => {
  if (item.tag === 1) {
    return request.defaults.baseURL + "/system/whuarchi/download/" + item.archiCover;
  } else {
    return request.defaults.baseURL + "/system/architecture/download/" + item.archiCover;
  }
}

const handleImagePreview = (imageUrl) => {
  data.currentImage = getAvatar(imageUrl); // 设置当前图片地址
  data.dialogVisible = true; // 显示图片预览弹窗
}

const loadWhu = () => {
  data.dialogWhuVisible = true;
  router.push('/front/archive/whu')
}
</script>