<template>
  <view>
    <!-- 这是自定义的组件 -->
    <!-- <my-search :bgcolor="'pink'" :radius="15"></my-search> -->
    <my-search @click="gotoSerach"></my-search>
      
    
    <view class="scroll-view-container">
      <!-- 左侧的滚动视图区域 -->
      <scroll-view class="left-scroll-view" scroll-y :style="{height: wh + 'px'}">
        <block v-for="(item, i) in cateList" :key="i">
          <view :class="['left-scroll-view-item', i === active ? 'active' : '']" @click="activeChanged(i)">{{item.cat_name}}</view>
        </block>
      </scroll-view>
      
      <!-- 右侧的滚动视图区域 -->
      <scroll-view class="right-scroll-view" scroll-y :style="{height: wh + 'px'}" :scrollTop='scrollTop'>
        <view class="cate-lv2" v-for="(item2, i2) in cateLevel2" :key="i2">
          <view class="cate-lv2-title">/ {{item2.cat_name}} /</view>
          <!-- 动态渲染三级分类的列表数据 -->
          <view class="cate-lv3-list">
            <!-- 三级分类 Item 项 -->
            <view class="cate-lv3-item" v-for="(item3, i3) in item2.children" :key="i3" @click="gotoGoodsList(item3)">
              <!-- 图片 -->
              <image :src="item3.cat_icon.replace('dev','web')"></image>
              <!-- 文本 -->
              <text>{{item3.cat_name}}</text>
            </view>
          </view>
        </view>
      </scroll-view>
    </view>
  </view>
</template>

<script>
 import badgeMix from '@/mixins/tabbar-badge.js'
 export default {
   // 将 badgeMix 混入到当前的页面中进行使用
   mixins: [badgeMix],
    data() {
      return {
        //当前设备可用的高度
        wh: 0,
        // 分类数据列表
        cateList: [],
        active: 0,
        // 二级分类列表
        cateLevel2: [],
        scrollTop: 0
      };
    },
    onLoad() {
      //获取设备信息
      const sysInfo = uni.getSystemInfoSync()
      console.log(sysInfo)
      this.wh = sysInfo.windowHeight
      
      // 调用获取分类列表数据的方法
      this.getCateList()
      
    },
    methods: {
      async getCateList() {
        // 发起请求
        const { data: res } = await uni.$http.get('https://api-ugo-web.itheima.net/api/public/v1/categories')
        console.log(res)
        // 判断是否获取失败
        if (res.meta.status !== 200) return uni.$showMsg()
        // 转存数据
        this.cateList = res.message
        
        // 为二级分类赋值
        this.cateLevel2 = res.message[0].children
      },
      
      activeChanged(i){
        this.active = i
        
        // 重新为二级分类赋值
        this.cateLevel2 = this.cateList[i].children
        //切换跳到顶部
        this.scrollTop = this.scrollTop === 0 ? 1 : 0
      },
      
      // 点击三级分类项跳转到商品列表页面
      gotoGoodsList(item3) {
        uni.navigateTo({
          url: '/subpkg/goods_list/goods_list?cid=' + item3.cat_id
        })
      },
      
      gotoSerach(){
        uni.navigateTo({
          url:'/subpkg/search/search'
        })
      }
      
    }
  }
</script>

<style lang="scss">
  .scroll-view-container{
    display: flex;
    .left-scroll-view{
      width: 240rpx;
      .left-scroll-view-item{
        line-height: 60px;
        background-color: #f7f7f7;
        text-align: center;
        font-size: 12px;
        
        &.active{
          background-color: #FFFFFF;
          position: relative;
          &::before{
            content:'';
            display: block;
            width: 3px;
            height: 30px;
            background-color: #C00000;
            position: absolute;
            top:50%;
            left:0;
            transform: translateY(-50%);
            
          }
        }
      }
    }
    
    .cate-lv2-title {
      font-size: 12px;
      font-weight: bold;
      text-align: center;
      padding: 15px 0;
    }
    .cate-lv3-list {
      display: flex;
      flex-wrap: wrap;
    
      .cate-lv3-item {
        width: 33.33%;
        margin-bottom: 10px;
        display: flex;
        flex-direction: column;
        align-items: center;
        image {
          width: 60px;
          height: 60px;
        }
        text {
          font-size: 12px;
        }
      }
    }
  }
</style>
