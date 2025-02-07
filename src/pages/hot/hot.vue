<script setup lang="ts">
import { onLoad } from '@dcloudio/uni-app'
import { getHotRecommendAPI } from '../../services/hot'
import type { HotResult, SubTypeItem, HotParams } from '@/types/hot'
import { ref } from 'vue'
// 热门推荐页 标题和url
const hotMap = [
  { type: '1', title: '特惠推荐', url: '/hot/preference' },
  { type: '2', title: '爆款推荐', url: '/hot/inVogue' },
  { type: '3', title: '一站买全', url: '/hot/oneStop' },
  { type: '4', title: '新鲜好物', url: '/hot/new' },
]

// 页面参数
const query = defineProps<{
  type: string
}>()
// 请求参数
const hotParams = ref<HotParams>({
  page: 31,
  pageSize: 10,
  subType: '',
})
const currentHot = hotMap.find((item) => item.type === query.type)
// 动态设置标题
uni.setNavigationBarTitle({ title: currentHot!.title })

// 获取热门推荐数据
const hotRecommend = ref<HotResult>()
const subTypes = ref<SubTypeItem[]>()
const activeIndex = ref(0)
const getHotRecommendData = async () => {
  const res = await getHotRecommendAPI(currentHot!.url, hotParams.value)
  hotRecommend.value = res.result
  subTypes.value = res.result.subTypes
}
// 页面初始化
onLoad(() => {
  console.log('热门推荐页初始化')
  getHotRecommendData()
})

// 结束标识
const finish = ref(false)

// 触底加载
const onScrolltolower = async () => {
  console.log('触底加载')
  const currentSubType = subTypes.value?.[activeIndex.value]
  console.log('currentSubType: ', currentSubType)
  const currentSubTypeId = currentSubType?.id
  if (currentSubType!.goodsItems.pages > currentSubType!.goodsItems.page) {
    // 页码累加
    hotParams.value.subType = currentSubType!.id
    hotParams.value.page = currentSubType!.goodsItems.page++
    const res = await getHotRecommendAPI(currentHot!.url, hotParams.value)
    currentSubType?.goodsItems.items.push(
      ...res.result.subTypes[activeIndex.value].goodsItems.items,
    )
  } else {
    finish.value = true
    return uni.showToast({
      title: '没有更多数据了',
      icon: 'none',
    })
  }
}
</script>

<template>
  <view class="viewport">
    <!-- 推荐封面图 -->
    <view class="cover">
      <image :src="hotRecommend?.bannerPicture"></image>
    </view>
    <!-- 推荐选项 -->
    <view class="tabs">
      <text
        class="text"
        :class="{ active: activeIndex === index }"
        @tap="activeIndex = index"
        v-for="(item, index) in subTypes"
        :key="item.id"
        >{{ item.title }}</text
      >
    </view>
    <!-- 推荐列表 -->
    <scroll-view
      scroll-y
      class="scroll-view"
      @scrolltolower="onScrolltolower"
      v-show="activeIndex === index"
      v-for="(item, index) in subTypes"
      :key="item.id"
    >
      <view class="goods">
        <navigator
          hover-class="none"
          class="navigator"
          v-for="goods in item.goodsItems.items"
          :key="goods.id"
          :url="`/pages/goods/goods?id=`"
        >
          <image class="thumb" :src="goods.picture"></image>
          <view class="name ellipsis">{{ goods.name }}</view>
          <view class="price">
            <text class="symbol">¥</text>
            <text class="number">{{ goods.price }}</text>
          </view>
        </navigator>
      </view>
      <view class="loading-text"> {{ finish ? '没有更多数据~' : '正在加载...' }}</view>
    </scroll-view>
  </view>
</template>

<style lang="scss">
page {
  height: 100%;
  background-color: #f4f4f4;
}

.viewport {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 180rpx 0 0;
  position: relative;
}

.cover {
  width: 750rpx;
  height: 225rpx;
  border-radius: 0 0 40rpx 40rpx;
  overflow: hidden;
  position: absolute;
  left: 0;
  top: 0;
}

.scroll-view {
  flex: 1;
}

.tabs {
  display: flex;
  justify-content: space-evenly;
  height: 100rpx;
  line-height: 90rpx;
  margin: 0 20rpx;
  font-size: 28rpx;
  border-radius: 10rpx;
  box-shadow: 0 4rpx 5rpx rgba(200, 200, 200, 0.3);
  color: #333;
  background-color: #fff;
  position: relative;
  z-index: 9;

  .text {
    margin: 0 20rpx;
    position: relative;
  }

  .active {
    &::after {
      content: '';
      width: 40rpx;
      height: 4rpx;
      transform: translate(-50%);
      background-color: #27ba9b;
      position: absolute;
      left: 50%;
      bottom: 24rpx;
    }
  }
}

.goods {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  padding: 0 20rpx 20rpx;

  .navigator {
    width: 345rpx;
    padding: 20rpx;
    margin-top: 20rpx;
    border-radius: 10rpx;
    background-color: #fff;
  }

  .thumb {
    width: 305rpx;
    height: 305rpx;
  }

  .name {
    height: 88rpx;
    font-size: 26rpx;
  }

  .price {
    line-height: 1;
    color: #cf4444;
    font-size: 30rpx;
  }

  .symbol {
    font-size: 70%;
  }

  .decimal {
    font-size: 70%;
  }
}

.loading-text {
  text-align: center;
  font-size: 28rpx;
  color: #666;
  padding: 20rpx 0 50rpx;
}
</style>
