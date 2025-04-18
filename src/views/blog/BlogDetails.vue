<template>
  <!-- 页头 -->
  <HeaderBar></HeaderBar>
  <div class="blog-details">
    <!-- 页头封面 -->
    <HeaderCover>
      <div class="blog-info">
        <h1 class="blog-title">
          {{ blogDetails.title }}
        </h1>
        <div class="blog-meta-data-wrap">
          <span class="blog-meta-data">
            发表于 {{ blogDetails.createTime }}
          </span>
          <span class="blog-meta-data-divider">|</span>
          <span class="blog-meta-data">
            分类 {{ blogDetails.categoryName }}
          </span>
        </div>
      </div>
    </HeaderCover>

    <div class="container">
      <!-- 侧边栏 -->
      <SideBar>
        <div class="sticky-layout" v-if="blogLoaded">
          <SideBarCatalog></SideBarCatalog>
          <BlogSeries
            :category="blogDetails.categoryName"
            :serial-no="blogDetails.serialNo"
          ></BlogSeries>
        </div>
      </SideBar>

      <!-- 文章内容 -->
      <div class="post-body">
        <div class="blog-content" v-html="blogDetails.content"></div>

        <!-- 版权声明 -->
        <div class="blog-signature">
          <img :src="author.avatar" alt="头像" />
          <div class="copyright">
            <div class="copyright-item">
              <span class="copyright-title">文章作者：</span>
              <span class="copyright-content">
                <router-link to="/"> {{ author.name }}</router-link></span
              >
            </div>
            <div class="copyright-item">
              <span class="copyright-title">文章链接：</span>
              <span class="copyright-content">
                <a :href="blogUrl">{{ blogUrl }}</a>
              </span>
            </div>
            <div class="copyright-item">
              <span class="copyright-title">版权声明：</span>
              <span class="copyright-content">
                本博客所有文章除特别声明外，均采用
                <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/"
                  >BY-NC-SA</a
                >
                许可协议。转载请注明出处！
              </span>
            </div>
          </div>
        </div>

        <!-- 标签 -->
        <div class="blog-tags" v-if="blogDetails.tags">
          <span> 标签： </span>
          <div v-for="tag in blogDetails.tags" :key="tag" class="tag-link">
            {{ tag }}
          </div>
        </div>

        <!-- 上一篇和下一篇 -->
        <div class="previous-next-blog">
          <div
            class="previous-blog"
            v-if="previousBlog.id"
            :style="{ width: nextBlog.id ? '50%' : '100%' }"
          >
            <router-link :to="`/blog/${previousBlog.id}`">
              <div class="previous-blog-info">
                <div class="label">« 上一篇</div>
                <div class="title">
                  {{ previousBlog.title }}
                </div>
              </div>
            </router-link>
          </div>

          <div
            class="next-blog"
            v-if="nextBlog.id"
            :style="{ width: previousBlog.id ? '50%' : '100%' }"
          >
            <router-link :to="`/blog/${nextBlog.id}`">
              <div class="next-blog-info">
                <div class="label">下一篇 »</div>
                <div class="title">
                  {{ nextBlog.title }}
                </div>
              </div>
            </router-link>
          </div>
        </div>
      </div>
    </div>

    <!-- 回到顶部 -->
    <BackToTop :cls="'blog-details'"></BackToTop>
    <!-- 页脚 -->
    <FootBar :style="{ position: 'relative', bottom: '60px' }"></FootBar>
  </div>
</template>

<script setup>
import { useStore } from "vuex";
import { reactive, ref, onMounted, nextTick, computed } from "vue";

import markdownIt from "../../utils/markdown-it";
import buildCodeBlock from "../../utils/code-block.js";
import router from "../../router";

import HeaderBar from "../../components/HeaderBar.vue";
import HeaderCover from "../../components/HeaderCover.vue";
import SideBar from "../../components/SideBar.vue";
import SideBarCatalog from "../../components/SideBarCatalog.vue";
import BlogSeries from "./BlogSeries.vue";
import FootBar from "../../components/FootBar.vue";
import BackToTop from "../../components/BackToTop.vue";
import { getBlogMdData } from "../../api/get.js";

const props = defineProps({
  id: {
    type: String,
    require: false,
    default: "",
  },
});

const store = useStore();

const archives = computed(() => store.state.blogsAbout.archives);
const categories = computed(() => store.state.blogsAbout.categories);
const author = computed(() => store.state.websiteAbout.author);

// 标识文章是否加载完成
const blogLoaded = ref(false);
// 当前文章的 URL
const blogUrl = ref(window.location.href);
// 上一篇文章信息
const previousBlog = reactive({});
// 下一篇文章信息
const nextBlog = reactive({});
// 文章内容的变量
const blogDetails = reactive({ createTime: "" }); // 初始化文章详情

// 渲染界面
onMounted(async () => {
  // 获取指定 ID 的文章内容数据
  const mdData = await getBlogMdData(props.id);

  if (!mdData) {
    router.push("/404");
    return;
  }

  // 根据文章 ID 在归档数据中查找对应的文章对象
  const blogObj = archives.value.find((item) => item.id === props.id);
  if (!blogObj) {
    router.push("/404");
    return;
  }

  // 将 Markdown 数据渲染为 HTML 并赋值到文章详情
  blogDetails.content = markdownIt.render(mdData);

  nextTick(() => {
    // 构建代码块高亮
    buildCodeBlock(".blog-content");
    // 标记文章加载完成
    blogLoaded.value = true;
  }).then(() => {
    // 设置文章的其他详情信息
    blogDetails.title = blogObj.title; // 设置文章标题
    blogDetails.createTime = blogObj.date; // 设置创建时间
    blogDetails.categoryName = blogObj.category; // 设置分类名称
    blogDetails.tags = blogObj.tags; // 设置标签
    blogDetails.serialNo = blogObj.serialNo; // 设置标签序号
  });

  // 更新上一篇和下一篇文章信息
  // 获取当前文章所属分类
  const category = categories.value[blogObj.category];
  if (typeof category !== "object") return;

  // 当前文章的序列号
  const serialNo = blogObj.serialNo;
  // 当前分类下的文章数量
  const categoryLength = category?.length;

  if (serialNo - 1 >= 0) {
    const previousObj = category[serialNo - 1];
    previousBlog.id = previousObj.id;
    previousBlog.title = previousObj.title;
  }

  if (serialNo + 2 <= categoryLength) {
    const nextObj = category[serialNo + 1];
    nextBlog.id = nextObj.id;
    nextBlog.title = nextObj.title;
  }
});
</script>

<style lang="less" scoped>
.blog-details {
  height: 100%;
  overflow-y: auto;
}
.container {
  padding: 40px 15px;
  max-width: 1300px;
  margin: 0 auto;
  display: flex;
  animation: fadeInUp 1s;
}

.wife-cover {
  display: flex;
  align-items: center;
  justify-content: center;
}

.blog-info {
  text-align: center;
  position: absolute;
  width: 100%;
  text-shadow: 0 3px 6px rgba(0, 0, 0, 0.3);
}

.blog-title {
  font-size: 35px;
  font-weight: normal;
  color: #222325;
  line-height: 1.5;
  margin-bottom: 15px;
  padding: 0 30px;
  overflow: hidden;
  display: -webkit-box;
  text-overflow: ellipsis;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}

.edit-icon {
  display: inline-block;
  cursor: pointer;
  transition: all 0.4s;

  &:hover {
    color: #ff7242;
  }
}

.blog-meta-data-wrap {
  display: flex;
  justify-content: center;
  color: #222325;
}

.blog-meta-data {
  font-size: 14px;
  box-sizing: border-box;
  line-height: 24px;
  overflow: hidden;
  -webkit-line-clamp: 1;
  display: -webkit-box;
  -webkit-box-orient: vertical;
}

.blog-meta-data-divider {
  color: #222325;
  margin: 3px 8px;
  font-size: 14px;
}

.post-body {
  width: 74%;
  background: white;
  border-radius: 8px;
  box-shadow: var(--card-box-shadow);
  padding: 30px 40px;
  box-sizing: border-box;

  :deep(.blog-content) {
    img {
      display: block;
      margin: 15px auto 15px;
      border-radius: 6px;
      width: 100%;
      cursor: pointer;
      cursor: zoom-in;
      box-shadow:
        0 1px 15px rgba(27, 31, 35, 0.15),
        0 0 1px rgba(106, 115, 125, 0.35);
    }

    h1 code,
    h2 code,
    h3 code,
    h4 code,
    h5 code,
    h6 code,
    p > code,
    li > code,
    table code {
      color: #c7254e;
      line-height: 1.2;
      font-family: consolas !important;
      vertical-align: middle;
      margin: 0 3px;
      background-color: #f9f2f4 !important;
      font-size: 14px !important;
      padding: 0.2em 0.3em !important;
      border-radius: 3px !important;
      border: 1px solid #f9f2f4 !important;
    }

    p {
      color: var(--text-color);
      font-size: 15px;
      line-height: 28px;
    }

    h1,
    h2,
    h3,
    h4,
    h5,
    h6 {
      overflow: hidden;
      -webkit-line-clamp: 4;
      text-overflow: ellipsis;
      display: -webkit-box;
      -webkit-box-orient: vertical;
      color: #1f2d3d;
      transition: all 0.2s ease-out;
    }

    h4,
    h5,
    h6 {
      font-size: 16px;
    }

    h1 {
      font-size: 24px;
      margin: 10px 0;
    }

    h2 {
      font-size: 20px;
    }

    h3 {
      font-size: 17px;
    }

    /* 代码样式 */
    pre {
      white-space: pre;
      position: relative;
      border-radius: 7px;
      color: #bababa;
      background-color: #282c34;
      font-size: 14px;
      padding: 0;

      code {
        border: none;
        border-radius: 7px;
        font-family:
          Consolas, Monaco, "Andale Mono", "Ubuntu Mono", monospace !important;
        line-height: 21px;
      }
    }

    kbd {
      background-color: #f7f7f7;
      color: #222325;
      border-radius: 0.25rem;
      border: 1px solid #cbcccd;
      box-shadow: 0 2px 0 1px #cbcccd;
      cursor: default;
      font-family: Arial, sans-serif;
      font-size: 0.75em;
      line-height: 1;
      min-width: 0.75rem;
      padding: 2px 5px;
      position: relative;
      top: -1px;

      &:hover {
        box-shadow: 0 1px 0 0.5px #cbcccd;
        top: 1px;
      }
    }

    a {
      color: #2d8cf0;
      text-decoration: none;
      transition: all 0.3s ease;
      position: relative;

      &::after {
        content: "";
        display: block;
        width: 0;
        height: 1px;
        position: absolute;
        left: 0;
        bottom: -2px;
        background: #2d8cf0;
        transition: all 0.3s ease-in-out;
      }

      &:hover::after {
        width: 100%;
      }
    }

    hr {
      position: relative;
      margin: 20px 0;
      border: 2px dashed #bfe4fb;
      width: 100%;
      box-sizing: content-box;
      height: 0;
      overflow: visible;
      box-sizing: border-box;
    }

    hr::before {
      position: absolute;
      top: -11px;
      left: 2%;
      z-index: 1;
      color: #bfe4fb;
      content: "✂";
      font-size: 21px;
      line-height: 1;
      -webkit-transition: all 1s ease-in-out;
      -moz-transition: all 1s ease-in-out;
      -o-transition: all 1s ease-in-out;
      -ms-transition: all 1s ease-in-out;
      transition: all 1s ease-in-out;
    }

    hr:hover::before {
      left: calc(98% - 20px);
    }

    table {
      font-size: 15px;
      width: 100%;
      margin: 15px 0px;
      display: block;
      overflow-x: auto;
      border: none;
      border-collapse: collapse;
      border-spacing: 0;

      &::-webkit-scrollbar {
        height: 4px !important;
      }

      th {
        background: #bfe4fb;
        border: 1px solid #a6d6f5;
        white-space: nowrap;
        font-weight: 400;
        padding: 6px 15px;
        min-width: 100px;
      }

      td {
        border: 1px solid #a6d6f5;
        padding: 6px 15px;
        min-width: 100px;
      }
    }

    ul,
    ol {
      li {
        margin: 4px 0px;
      }
    }

    ul li {
      list-style: circle;

      &::marker {
        transition: all 0.4s;
        /* color: #49b1f5; */
        color: var(--theme-color);
        font-weight: 600;
        font-size: 1.05em;
      }

      &:hover::marker {
        color: #ff7242;
      }
    }

    blockquote {
      border: none;
      margin: 15px 0px;
      color: inherit;
      border-radius: 4px;
      padding: 1px 15px;
      border-left: 4px solid var(--theme-color);
      background-color: #f8f8f8;
    }
  }

  .blog-signature {
    border: 1px solid #ddd;
    position: relative;
    overflow: hidden;
    margin: 30px 5px 10px 5px;
    border-radius: 6px;
    display: flex;
    align-items: center;
    padding: 12px;
    transition: all 0.4s;

    &:hover {
      box-shadow: 0 3px 15px rgba(0, 0, 0, 0.1);
      transform: translateY(-1px);
    }

    img {
      margin-left: 10px;
      width: 90px;
      height: 90px;
      border-radius: 50%;
      transition: all 0.6s;

      &:hover {
        transform: rotate(-360deg);
      }
    }

    .copyright {
      padding-left: 20px;

      .copyright-item {
        display: -webkit-box;
        overflow: hidden;
        text-overflow: ellipsis;
        -webkit-line-clamp: 1;
        -webkit-box-orient: vertical;
        line-height: 28px;
        font-size: 15px;
        color: var(--text-color);

        a {
          color: #99a9bf;
          transition: all 0.4s;
          cursor: pointer;

          &:hover {
            color: #19b1f5;
          }
        }

        .copyright-title {
          font-weight: bold;
          color: #19b1f5;
        }
      }
    }
  }

  .blog-tags {
    padding-left: 3px;
    margin-top: 20px;
    color: var(--text-color);
    font-size: 15px;
    display: flex;
    gap: 8px;
    align-items: center;

    .tag-link {
      background-color: #4a87ed;
      padding: 4px;
      color: #fff;
      border-radius: 4px;
    }

    a {
      border-radius: 4px;
      font-size: 13px;
      padding: 3px 12px;
      text-decoration: none;
      transition: all 0.4s;
      background: #49b1f5;
      margin-right: 8px;
      color: white;
      display: inline-block;
    }
  }

  .previous-next-blog {
    width: 100%;
    margin-top: 50px;
    overflow: hidden;
    background: black;
    display: flex;
    border-radius: 9px;

    .previous-blog,
    .next-blog {
      width: 50%;

      a {
        height: 90px;
        overflow: hidden;
        display: block;
        position: relative;

        img {
          height: 100%;
          width: 100%;
          position: absolute;
          object-fit: cover;
          opacity: 0.5;
          transition: all 0.6s ease-in-out;

          &:hover {
            transform: scale(1.1);
            opacity: 0.8;
          }
        }

        .previous-blog-info,
        .next-blog-info {
          pointer-events: none;
          position: absolute;
          top: 50%;
          width: 100%;
          padding: 20px 40px;
          transform: translate(0, -50%);
          color: white;
          line-height: 28px;
          box-sizing: border-box;

          .label {
            font-size: 13px;
          }

          .title {
            font-weight: 500;
            font-size: 14px;
            display: -webkit-box;
            overflow: hidden;
            text-overflow: ellipsis;
            -webkit-line-clamp: 1;
            -webkit-box-orient: vertical;
          }
        }

        .next-blog-info {
          text-align: right;
        }
      }
    }
  }

  #blog-desc {
    font-size: 14px;
    margin-top: 8px;
    text-align: right;
    color: var(--text-color);

    span,
    a {
      padding: 0 4px;
    }

    a {
      text-decoration: none;
      color: var(--text-color);
      transition: all 0.6s ease-out;

      &:hover {
        color: var(--theme-color);
      }
    }
  }

  #comment-area {
    margin-top: 70px;

    .comment-area-title {
      font-size: 20px;
      margin: 20px 0;
      padding-bottom: 5px;
      color: var(--text-color);

      .comment-icon {
        margin-right: 7px;
        color: #e173b3;
      }
    }

    :deep(#comment-pagination) {
      margin: 20px 0;
      justify-content: center;

      .number,
      .btn-prev,
      .btn-next {
        border-radius: 6px;
      }
    }
  }

  #comment-form {
    #comment-form-title {
      font-size: 20px;
      margin: 40px 0 20px;
      color: var(--text-color);
    }

    #comment-editor {
      #mavon {
        border-color: #eef2f8;
      }

      #comment-submit-btn {
        color: white;
        background-color: var(--theme-color);
        border: 1px solid var(--theme-color);
        border-radius: 5px;
        cursor: pointer;
        padding: 7px 17px;
        font-size: 13px;
        margin: 10px 0;
        transition: all 0.3s ease-out;

        &:hover {
          opacity: 0.7;
        }
      }
    }
  }
}

.sticky-layout {
  position: sticky;
  top: 20px;
}

@media screen and (max-width: 900px) {
  .post-body {
    width: 100%;
  }

  #blog-desc {
    display: none;
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
