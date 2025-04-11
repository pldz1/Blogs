<template>
  <!-- 页头 -->
  <HeaderBar></HeaderBar>
  <div class="home">
    <div class="container"></div>

    <!-- 页脚 -->
    <FootBar></FootBar>

    <!-- 滚动到顶部按钮 -->
    <BackToTop :cls="'home'" ref="backToTopRef"></BackToTop>
  </div>
</template>

<script setup>
import { useStore } from "vuex";
import { ref } from "vue";

import HeaderBar from "@/components/HeaderBar.vue";
import FootBar from "@/components/FootBar.vue";
import BackToTop from "@/components/BackToTop.vue";

const backToTopRef = ref(null);
const store = useStore();

// 获取博客分类数据
const categories = store.state.blogsAbout.categories;
const categoryList = Object.keys(categories).map((key) => ({
  name: key,
  count: categories[key]?.length || 0,
  isShowCount: true,
}));
const tags = store.state.blogsAbout.tags;

// 显示内容的响应式数据
const blogsCategroyList = ref([]);

// 通用处理函数
const updateBlogList = (sourceMap, key) => {
  blogsCategroyList.value = (sourceMap[key] || []).map((blog) => ({
    ...blog,
    url: `/blog/${blog.id}`,
  }));

  backToTopRef.value?.scrollToTop();
};

// 显示分类下的文章
const handleChangeCategory = (categoryName) => {
  updateBlogList(categories, categoryName);
};

// 显示标签下的文章
const handleChangeTag = (tagName) => {
  updateBlogList(tags, tagName);
};
</script>

<style scoped>
.home {
  height: 100%;
  width: 100%;
  overflow-y: auto;
}

.container {
  padding: 40px 15px;
  max-width: 1300px;
  margin: 0 auto;
  display: flex;
  animation: fadeInUp 1s;
}

.home-content {
  width: 74%;
  display: flex;
  justify-content: flex-start;
  flex-direction: column;
  align-items: center;
  min-height: 600px;
}
</style>
