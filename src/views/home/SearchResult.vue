<template>
  <div class="search-main" @scroll="onScroll" ref="searchMain">
    <a-list class="search-list" :split="false">
      <template v-for="item in state.searchList">
        <a-list-item v-if="item.title" :key="item.url">
          <a-card style="width: 100%">
            <a-card-meta :description="item.urlText">
              <template #title>
                <a class="link-title" :href="item.url" :target="state.openPageTarget">
                  {{ item.title }}
                </a>
              </template>
              <template #avatar v-if="!isEmpty(item.icon)">
                <a-avatar :src="item.icon" />
              </template>
            </a-card-meta>

            <a-divider style="margin: 16px 0" />
            <div class="item-content">
              <img v-if="item.cover" class="content-cover" :src="item.cover" />
              {{ item.description }}
            </div>
          </a-card>
        </a-list-item>
      </template>

      <a-list-item class="load-more">
        <a-skeleton :loading="state.loading" active avatar />
      </a-list-item>
    </a-list>
  </div>
</template>

<script lang="ts" setup>
import { useStore } from "@/store"
import { SearchResultData } from "@/types"
import { isEmpty } from "@/utils/common"
import { computed, onMounted, reactive, ref } from "vue"
import { useRoute } from "vue-router"

const route = useRoute(),
  { state: stateX } = useStore(),
  { text } = route.params

const state = reactive({
  page: 0,
  loading: false,
  searchList: [] as SearchResultData,
  currentRule: computed(() => stateX.search.rules[stateX.setting.search.currentEngine]),
  openPageTarget: computed(() => stateX.setting.search.openPageTarget)
})

async function loadSearchList() {
  state.loading = true
  const searchList = await state.currentRule.getSearchList(text as string, state.page)

  console.log("searchList:", searchList)
  state.searchList = state.searchList.concat(searchList)
  state.loading = false
}

const searchMain = ref<Element>()
function onScroll() {
  const { scrollHeight, clientHeight, scrollTop } = searchMain.value!

  if (scrollHeight - (clientHeight + scrollTop) <= 1 && !state.loading) {
    state.page += 1
    loadSearchList()
  }
}

onMounted(loadSearchList)
</script>

<style lang="less">
.search-main {
  position: fixed;
  height: 100%;
  width: 100%;
  overflow: auto;
  display: flex;
  justify-content: center;

  .search-list {
    width: 750px;
    padding-top: 96px;

    .link-title {
      font-size: 20px;
    }

    .load-more {
      height: 128px;
    }

    .item-content {
      img {
        width: 128px;
        float: left;
        margin-right: 8px;
      }
    }
  }
}
</style>
