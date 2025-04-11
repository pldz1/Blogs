<template>
  <div :class="cardClass">
    <!-- 缩略图 -->
    <router-link :to="`${blog.url}`" :class="thumbailLinkClass"
      ><img
        :src="blog.thumbnail"
        @error.once=""
        alt="缩略图"
        class="post-blog-thumbnail"
      />
    </router-link>

    <!-- 卡片信息 -->
    <div class="post-blog-info">
      <!-- 显示卡片标题 -->
      <router-link :to="`${blog.url}`" class="post-blog-title"
        >{{ blog.title }}
      </router-link>

      <!-- 基本数据 -->
      <div class="post-blog-meta-data-wrap">
        <span class="post-blog-meta-data" v-if="blog.date">
          <div class="icon-16" v-html="calendar16"></div>
          创建于 {{ blog.date }}
        </span>
      </div>

      <!-- 摘要 -->
      <div v-if="ctype == 'codespace'" class="post-blog-summary">
        {{ `体验 🚀: ${blog.summary}` }}
      </div>
      <div v-else class="post-blog-summary">
        {{ `摘要 📃: ${blog.summary}` }}
      </div>

      <div
        v-if="ctype == 'codespace'"
        v-html="codespace32"
        class="icon-32-a post-blog-type-icon"
      ></div>
      <div v-else v-html="blog32" class="icon-32-a post-blog-type-icon"></div>
    </div>
  </div>
</template>

<script setup>
import { reactive } from "vue";
import { calendar16 } from "@/assets/svg";
import { blog32, codespace32 } from "@/assets/svg";
// Props 接收
const props = defineProps({
  blog: {
    type: Object,
    require: true,
  },
  ctype: {
    type: String,
    require: true,
  },
  reverse: {
    type: Boolean,
    require: false,
  },
});

// 响应式类名
const cardClass = reactive(["home-card"]);
const thumbailLinkClass = reactive(["post-blog-thumbail-link"]);

if (props.reverse) {
  cardClass.push("home-card-reversed");
  thumbailLinkClass.push("post-blog-thumbail-link-reversed");
}
</script>
