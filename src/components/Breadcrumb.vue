<template>
  <el-breadcrumb class="app-breadcrumb" separator="/">
    <transition-group name="breadcrumb">
      <el-breadcrumb-item v-for="(item, index) in levelList" :key="item.path">
        <span
          v-if="item.redirect === 'noRedirect' || index == levelList.length - 1"
          class="no-redirect"
          >{{ item.meta.title }}</span
        >
        <a v-else @click.prevent="handleLink(item)">{{ item.meta.title }}</a>
      </el-breadcrumb-item>
    </transition-group>
  </el-breadcrumb>
</template>

<script lang="ts" setup>
import * as pathToRegexp from "path-to-regexp";
import { ref, watch } from "vue";
import { RouteLocationMatched, useRoute, useRouter } from "vue-router";
const route = useRoute();
const router = useRouter();

const levelList = ref<RouteLocationMatched[]>([]);

function getBreadcrumb() {
  // only show routes with meta.title
  let matched: any[] = route.matched.filter(
    (item) => item.meta && item.meta.title
  );
  const first = matched[0];

  if (!isDashboard(first)) {
    matched = [{ path: "/", meta: { title: "Home" } }].concat(matched);
  }

  levelList.value = matched.filter(
    (item) => item.meta && item.meta.title && item.meta.breadcrumb !== false
  );
}

function isDashboard(route: RouteLocationMatched) {
  const name: any = route?.name;
  if (!name) {
    return false;
  }
  return name.trim().toLocaleLowerCase() === "Home".toLocaleLowerCase();
}

function pathCompile(path: any) {
  // To solve this problem https://github.com/PanJiaChen/vue-element-admin/issues/561
  const { params } = router.currentRoute.value;
  var toPath = pathToRegexp.compile(path);
  return toPath(params);
}

function handleLink(item: { redirect: any; path: any }) {
  const { redirect, path } = item;
  if (redirect) {
    router.push(redirect);
    return;
  }
  router.push(pathCompile(path));
}
getBreadcrumb();

watch(
  route,
  () => {
    getBreadcrumb();
  },
  { deep: true }
);
</script>

<style lang="scss" scoped>
.app-breadcrumb.el-breadcrumb {
  display: inline-block;
  font-size: 14px;
  line-height: 50px;
  margin-left: 8px;

  .no-redirect {
    color: #97a8be;
    cursor: text;
  }
}
</style>
