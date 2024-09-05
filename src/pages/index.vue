<script setup lang="ts">
defineOptions({
  name: 'IndexPage',
})

const { t } = useI18n()

const searchValue = ref('')

const { isFetching: isTopFetching, error: topError, data: topData } = useFetch<any>('/api/rss/topgrossingapplications/limit=10/json', {
  timeout: 10 * 1000,
}).json()
const topList = computed(() => topData.value?.feed?.entry || [])

const { execute: executeList, isFetching: isListFetching, error: listError, data: listData } = useFetch<any>('/api/rss/topfreeapplications/limit=100/json', {
  immediate: false,
  timeout: 10 * 1000,
}).json()
const list = computed(() => listData.value?.feed?.entry || [])

const { execute: executeLookup, data: lookupData } = useFetch<any>(computed(() => `/api/lookup?id=${list.value?.map((item: any) => item.id.attributes['im:id']).join(',')}`), {
  immediate: false,
  timeout: 20 * 1000,
}).json()
const lookupMap = computed(() => {
  const map: Record<string, any> = {}
  lookupData.value?.results?.forEach((item: any) => {
    map[item.trackId] = item
  })

  return map
})

const searchList = ref<any[]>([])

function search() {
  if (searchValue.value) {
    searchList.value = list.value.filter((item: any) =>
      item['im:name'].label.includes(searchValue.value)
      || item.category.attributes.label.includes(searchValue.value)
      || item['im:artist'].label.includes(searchValue.value))
  }
  else {
    searchList.value = list.value
  }
}

const debouncedSearch = useDebounceFn(search, 500)

async function getList() {
  await executeList()
  search()
  await executeLookup()
}

getList()
</script>

<template>
  <div>
    <header sticky top-0 z-999 bg-white px-2 py-3 shadow>
      <div w-full rounded-xl px-2 py-2 flex="~ gap-2 items-center" bg="[#F4F4F4]">
        <span i-carbon-search text="[#A0A9B0]" />
        <input
          v-model.trim="searchValue" :placeholder="t('search')" w-full bg-transparent outline-none
          class="placeholder-#A0A9B0" @input="debouncedSearch"
        >
      </div>
    </header>

    <section border-b px-2 py-3>
      <h2 text="xl" mb-2>
        推荐
      </h2>
      <template v-if="!isTopFetching">
        <div v-if="topList.length" flex="~ gap-5" overflow-x-auto px-2>
          <div v-for="item in topList" :key="item.id.label" w-25 text="center 15px">
            <img :src="item['im:image'][1].label" loading="lazy" mx-auto h-25 min-w-25 rounded-xl>
            <p truncate>
              {{ item['im:name'].label }}
            </p>
            <p truncate op70>
              {{ item.category.attributes.label }}
            </p>
          </div>
        </div>
        <p v-else text-center>
          暂无数据
        </p>
      </template>
      <p v-else-if="topError" text-center>
        加载失败
      </p>
      <p v-else text-center>
        加载中...
      </p>
    </section>

    <section px-4 py-3>
      <template v-if="!isListFetching">
        <template v-if="searchList.length">
          <div
            v-for="(item, index) in searchList" :key="item.id.label" text="15px" flex="~ items-center gap-4" w-full
            border-b py-4
          >
            <span text="base" op70>{{ index + 1 }}</span>
            <img
              :src="item['im:image'][1].label" loading="lazy" h-22 w-22
              :class="[index % 2 === 0 ? 'rounded-xl' : 'rounded-full']"
            >
            <div flex="~ col" h-22 w-0 flex-1>
              <p truncate>
                {{ item['im:name'].label }}
              </p>
              <p truncate op70>
                {{ item.category.attributes.label }}
              </p>
              <div mt-auto flex="~ items-center gap-2">
                <div relative inline-block>
                  <div flex>
                    <span v-for="i in 5" :key="i" i-carbon-star text="[#F3CB4C]" />
                  </div>
                  <div
                    absolute left-0 top-0 flex overflow-hidden :style="{
                      width: `${(lookupMap[item.id.attributes['im:id']]?.averageUserRating || 0) / 5 * 100}%`,
                    }"
                  >
                    <span v-for="i in 5" :key="i" i-carbon-star-filled text="[#F3CB4C]" flex-shrink-0 />
                  </div>
                </div>

                <span op70>({{ lookupMap[item.id.attributes['im:id']]?.userRatingCount || 0 }})</span>
              </div>
            </div>
          </div>
        </template>
        <p v-else text-center>
          暂无数据
        </p>
      </template>
      <p v-else-if="listError" text-center>
        加载失败
      </p>
      <p v-else text-center>
        加载中...
      </p>
    </section>
  </div>
</template>
