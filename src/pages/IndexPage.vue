<template>
  <div class="index-page">
    <!--    <a-input-search-->
    <!--      v-model:value="searchText"-->
    <!--      placeholder="input search text"-->
    <!--      enter-button="Search"-->
    <!--      size="large"-->
    <!--      @search="onSearch"-->
    <!--    />-->
    <a-auto-complete
      v-model:value="searchText"
      :options="options"
      style="width: 85%"
      placeholder="input here"
      @select="onSearch"
      @search="getSearchPrompt"
    />
    <a-button
      type="primary"
      style="left-margin: 10px; width: 15%"
      @click="onSearch(searchText)"
      >Primary Button
    </a-button>
    <my-divider />
    <a-tabs v-model:activeKey="activeKey" @change="onTabChange">
      <a-tab-pane key="post" tab="文章">
        <PostList :post-list="postList" />
      </a-tab-pane>
      <a-tab-pane key="picture" tab="图片">
        <PictureList :picture-list="pictureList" />
      </a-tab-pane>
      <a-tab-pane key="user" tab="用户">
        <UserList :user-list="userList" />
      </a-tab-pane>
      <a-tab-pane key="video" tab="视频">
        <VideoList :video-list="videoList" />
      </a-tab-pane>
    </a-tabs>
  </div>
</template>

<script setup lang="ts">
import PostList from "@/components/PostList.vue";
import UserList from "@/components/UserList.vue";
import VideoList from "@/components/VideoList.vue";
import { ref, watchEffect } from "vue";
import PictureList from "@/components/PictureList.vue";
import MyDivider from "@/components/MyDivider.vue";
import { useRoute, useRouter } from "vue-router";

import myAxios from "@/plugins/MyAxios";
import { message } from "ant-design-vue";

const route = useRoute();
const activeKey = route.params.category;
const router = useRouter();

const postList = ref([]);
const userList = ref([]);
const pictureList = ref([]);
const videoList = ref([]);
const options = ref<any[]>([]);

const initSearchParams = {
  type: activeKey,
  text: "",
  pageNum: 1,
  pageSize: 20,
};

const searchText = ref(route.query.text || "");

const loadDataOld = (params: any) => {
  const postQuery = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("/post/list/page/vo", postQuery).then((res: any) => {
    postList.value = res.records;
  });
  const pictureQuery = {
    ...params,
    searchText: params.text,
  };

  myAxios.post("/picture/list/page/vo", pictureQuery).then((res: any) => {
    pictureList.value = res.records;
  });

  const userQuery = {
    ...params,
    userName: params.text,
  };
  myAxios.post("/user/list/page/vo", userQuery).then((res: any) => {
    userList.value = res.records;
  });
};

/**
 * 聚合搜索
 * @param params
 */
const loadAllData = (params: any) => {
  const query = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("/search/all", query).then((res: any) => {
    pictureList.value = res.pictureList;
    postList.value = res.postList;
    userList.value = res.userList;
    videoList.value = res.videoList();
  });
};

/**
 * 加载单个数据
 * @param params
 */
const loadData = (params: any) => {
  const { type = "post" } = params;
  if (!type) {
    message.error("类别为空");
    return;
  }
  const query = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("/search/all", query).then((res: any) => {
    if (type === "post") {
      postList.value = res.dataList;
    } else if (type === "user") {
      userList.value = res.dataList;
    } else if (type === "picture") {
      pictureList.value = res.dataList;
    } else if (type === "video") {
      videoList.value = res.dataList;
    }
  });
};

const searchParams = ref(initSearchParams);

const onTabChange = (key: string) => {
  router.push({
    path: `/${key}`,
    query: searchParams.value,
  });
};

watchEffect(() => {
  searchParams.value = {
    ...initSearchParams,
    text: route.query.text,
    type: route.params.category,
  } as any;
  loadData(searchParams.value);
});
const onSearch = (value: string) => {
  router.push({
    query: {
      ...searchParams.value,
      text: value,
    },
  });
};
const getSearchPrompt = (value: string) => {
  options.value = [];
  if (value) {
    myAxios.get("search/getSearchPrompt?keyword=" + value).then((res: any) => {
      for (let i = 0; i < res.length; i++) {
        const tempMap = {
          value: res[i],
          color: "red",
        };
        options.value.push(tempMap);
      }
    });
  }
};
</script>
