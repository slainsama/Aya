<script lang="ts" setup>
// 导入配置文件
import { serverConfig } from "../config/Server";

// 导入定义
import { Product, ProductListResponse } from "../define/Product";

// 导入组件
import MarketCard from "../components/market/MarketCard.vue";
import Footer from "../components/Footer.vue";
import TopBar from "../components/TopBar.vue";

// 导入第三方组件
import { NGrid, NGridItem, NEmpty, NButton } from "naive-ui";
import { onMounted, ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import ProductDetail from "../components/market/ProductDetail.vue";

// 使用接口
const router = useRouter();

// 定义 ref 变量
const productList = ref<Product[]>([]); // 产品列表
const hasPriviledge = ref(false); // 判断是否有权限进行活动
const loginSuccess = ref(false); // 判断登录是否成功
const shouldShowDetail = ref(false); // 判断是否需要展示商品细节
const targetPID = ref("");

// 获取商品列表
async function getProducts(): Promise<Product[]> {
  const token = localStorage.getItem("token");
  if (token == null) {
    loginSuccess.value = false;
    return [];
  }

  const getProductsUrl =
    serverConfig.urlPrefix + serverConfig.apiMap.product.list;

  const respData = (
    await axios.get(getProductsUrl, {
      headers: { Authorization: token },
    })
  ).data as ProductListResponse;

  if (respData.code == 0) {
    hasPriviledge.value = true;
    loginSuccess.value = true;
    return respData.data;
  } else if (respData.code == 1) {
    // 登录成功但是没有权限
    hasPriviledge.value = false;
    loginSuccess.value = true;
  } else {
    // 登录没有成功
    hasPriviledge.value = false;
    loginSuccess.value = false;
  }

  return [];
}

// 页面跳转函数
function jumpToLogin() {
  router.push("/login");
}

// 挂载流程
onMounted(() => {
  (async () => {
    productList.value = await getProducts(); // 获取产品列表
  })();
});
</script>

<template>
  <TopBar />

  <!-- 没有登录显示的空状态 -->
  <n-empty
    v-if="!loginSuccess"
    :size="'huge'"
    id="not-online-state"
    description="需要登录开始秒杀"
  >
    <template #extra>
      <n-button
        color="rgb(193, 46, 50)"
        round
        id="login-button"
        :type="'default'"
        :size="'large'"
        @click="jumpToLogin"
      >
        点我登录
      </n-button>
    </template>
  </n-empty>

  <!-- 登录以后展现列表 -->
  <NGrid v-else-if="hasPriviledge" cols="5 1700:6">
    <NGridItem v-for="product in productList">
      <MarketCard
        :pid="product.pid"
        :begin-time="product.begin_time"
        :end-time="product.end_time"
        :name="product.name"
        :stock="product.stock"
        :price="product.price"
        :money-rate="product.money_rate"
        @show-detail="shouldShowDetail = true"
        v-model:targetPID="targetPID"
      />
    </NGridItem>
  </NGrid>

  <!-- 没有权限显示的空状态 -->
  <n-empty
    v-else
    :size="'huge'"
    id="not-online-state"
    description="你暂时没有资格参加秒杀, 请联系我们的客服"
  >
  </n-empty>

  <!-- 产品的细节组件 -->
  <ProductDetail :pid="targetPID" v-model:shouldShowDetail="shouldShowDetail" />

  <Footer />
</template>

<style>
#not-online-state {
  margin: 30vh auto;
  height: 100px;
}

#login-button {
  margin-top: 20px;
}
</style>
