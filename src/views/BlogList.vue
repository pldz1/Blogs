<template>
  <!-- 页头 -->
  <HeaderBar></HeaderBar>
  <div class="home">
    <div class="container">
      <!-- 侧边栏 -->
      <div class="side-content">
        <WebSiteAuthorCard></WebSiteAuthorCard>
        <SideBarCategory
          :title="`全部博客类别`"
          :category-list="categoryList"
          @on-change-category="handleChangeCategory"
        ></SideBarCategory>
        <SideBarTag @on-change-tag="handleChangeTag"></SideBarTag>
      </div>

      <!-- 界面的主题内容 -->
      <div class="home-content">
        <div class="home-card-list">
          <BlogCard
            v-for="blog in blogsCategroyList"
            ctype="blog"
            :key="blog.id"
            :blog="blog"
            :reverse="false"
          ></BlogCard>
        </div>
      </div>
    </div>

    <!-- 页脚 -->
    <FootBar></FootBar>

    <!-- 滚动到顶部按钮 -->
    <BackToTop :cls="'home'" ref="backToTopRef"></BackToTop>
  </div>
</template>

<script setup>
import { useStore } from "vuex";
import { onMounted, ref } from "vue";

import HeaderBar from "@/components/HeaderBar.vue";
import WebSiteAuthorCard from "@/components/WebSiteAuthorCard.vue";
import SideBarTag from "@/components/SideBarTag.vue";
import SideBarCategory from "@/components/SideBarCategory.vue";
import BlogCard from "@/components/BlogCard.vue";
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

/**
 *
 */
onMounted(() => {
  // 清空当前分类列表
  blogsCategroyList.value = [];

  for (const key in categories) {
    if (categories.hasOwnProperty(key)) {
      // 获取每个分类的第一个项目
      const serialNo0Item = categories[key][0];
      if (typeof serialNo0Item == "object") {
        blogsCategroyList.value.push({
          // 解构原始数据
          ...serialNo0Item,
          // 覆盖原来名字, 改成 👉 系列-名字
          title: `${serialNo0Item.category} - ${serialNo0Item.title}`,
          // 构建分类的URL路径
          url: `/blog/${serialNo0Item.id}`,
        });
      }
    }
  }
});
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

.home-card-list {
  width: 74%;
}

.home-card-list .home-card {
  margin-top: 20px;
}

.home-card-list .home-card:nth-child(1) {
  margin-top: 0;
}

.side-content {
  width: 26%;
  margin-right: 20px;
}

#pagination {
  margin-top: 20px;
  justify-content: center;
}

#pagination > button {
  box-shadow: var(--card-box-shadow);
  background: white;
  border-radius: 8px;
  height: 35px;
  width: 35px;
}

#pagination li {
  box-shadow: var(--card-box-shadow);
  background-color: white;
  border-radius: 8px;
  margin: 0 6px;
  height: 35px;
  width: 35px;
}

#pagination li.active {
  color: white;
  background: var(--theme-color);
  font-weight: normal;
}

@media screen and (max-width: 900px) {
  .side-content {
    display: none;
  }

  .home-content {
    width: 100%;
  }
}

@keyframes fadeInUp {
  from {
    margin-top: 50px;
    opacity: 0;
  }

  to {
    margin-top: 0;
    opacity: 1;
  }
}
</style>
