<template>
  <view>
    <scroll-view v-if="commentList.length === 0?false:true" :style="{'height':windowHeight+'px'}"
      @scrolltolower="currentChange" scroll-y="true">
      <view class="cu-list menu-avatar">
        <view class="cu-item" v-for="(item,index) in commentList" :key="index"
          @click="toMicroBlog(item.commented_blog.objectId)">
          <view class="cu-avatar round lg" :style="{'background-image':'url('+item.creator.avatarUrl+')'}"></view>
          <view class="content">
            <view class="text-black text-bold">{{item.creator.nickname}}</view>
            <view class="text-black text-xs">{{item.review}}</view>
            <view class="text-grey text-xs">{{item.createdAt}}</view>
          </view>
          <view class="action">
            <view class="blog-img" :style="{'background-image':'url('+item.commented_blog.imgList.split(',')[0]+')'}">
            </view>
          </view>
        </view>
      </view>
    </scroll-view>
    <view v-if="commentList.length === 0?true:false" class="flex justify-center align-center"
      :style="{'height':windowHeight+'px'}">
      <view style="text-align:center">
        <image src="http://static.xch752.com/undraw_Mobile_app_p3ts.png" mode="aspectFit"
          style="width: 200upx;height: 170upx;"></image>
        <view class="text-gray margin-top-sm">还没有收到评论</view>
      </view>
    </view>
  </view>
</template>

<script>
  import uniNavBar from "@/components/uni-nav-bar/uni-nav-bar.vue"
  export default {
    components: {
      uniNavBar
    },
    data() {
      return {
        // 评论分页
        pageNum: 1,
        pageSize: 20,
        // 登录者ID
        objectId: '',
        // 评论列表
        commentList: [],
        // scrollview高度
        windowHeight: ''
      }
    },
    onLoad: function (option) {
      this.loadUserData()
      this.getSysInfo()
      this.objectId = option.objectId
      this.initData()
    },
    methods: {
      // 获取屏幕高度
      getSysInfo() {
        uni.getSystemInfo({
          success: (e) => {
            console.log(e)
            this.windowHeight = e.windowHeight
          }
        })
      },
      // 获取用户信息
      loadUserData() {
        try {
          this.objectId = uni.getStorageSync('bmob').objectId
          if (!this.objectId) {
            uni.reLaunch({
              url: '../../Login/Login',
              success: () => {
                uni.showToast({
                  title: '登录过期',
                  duration: 2000,
                  icon: 'none'
                })
              }
            })
          }
        } catch (e) {
          // error
          console.log(e)
          uni.reLaunch({
            url: '../../Login/Login',
            success: () => {
              uni.showToast({
                title: '登录过期',
                duration: 2000,
                icon: 'none'
              })
            }
          })
        }
      },
      // 返回
      navBack() {
        uni.navigateBack({
          delta: 1
        })
      },
      // 初始化数据
      initData() {
        this.pageNum = 1
        const query = this.Bmob.Query('Comment')
        const query1 = query.equalTo('replyUser', '===', this.objectId)
        const query2 = query.equalTo('commented_creator', '===', this.objectId)
        query.include('commented_blog', 'creator', 'replyUser')
        query.limit(this.pageSize)
        query.or(query1, query2);
        query.order('-createdAt')
        uni.showLoading({
          title: '加载中'
        })
        query.find().then(res => {
          res.map(item => {
            item.createdAt = this.getDateDiff(item.createdAt)
          })
          uni.hideLoading()
          this.commentList = res
        }).catch(err => {
          uni.hideLoading()
          uni.showToast({
            title: `加载失败${err}`,
            duration: 2000,
            icon: 'none'
          })
        })
      },
      // 下拉加载
      currentChange() {
        const query = this.Bmob.Query('Comment')
        const query1 = query.equalTo('replyUser', '===', this.objectId)
        const query2 = query.equalTo('commented_creator', '===', this.objectId)
        query.include('commented_blog', 'creator', 'replyUser')
        query.limit(this.pageSize)
        query.or(query1, query2);
        query.skip(this.pageNum * this.pageSize)
        query.order('-createdAt')
        uni.showLoading({
          title: '加载中'
        })
        query.find().then(res => {
          uni.hideLoading()
          if (res.length == 0) {
            uni.showToast({
              title: '没有更多了',
              duration: 2000,
              icon: 'none'
            })
            return
          }
          res.map(item => {
            item.createdAt = this.getDateDiff(item.createdAt)
          })
          this.pageNum++
          this.commentList = this.commentList.concat(res)
        }).catch(err => {
          uni.hideLoading()
          uni.showToast({
            title: `加载失败${err}`,
            duration: 2000,
            icon: 'none'
          })
        })
      },
      toMicroBlog(objectId) {
        uni.navigateTo({
          url: `../../MicroBlog/MicroBlog?microBlogId=${objectId}`
        })
      },
      // 时间格式化几分钟前、几小时前
      // 时间戳转多少分钟之前
      getDateDiff(dateTimeStamp) {
        // 时间字符串转时间戳
        //#ifdef APP-PLUS
        var f = dateTimeStamp.split(' ', 2);
        var d = (f[0] ? f[0] : '').split('-', 3);
        var timestamp = new Date(
          parseInt(d[0], 10) || null,
          (parseInt(d[1], 10) || 1) - 1,
          parseInt(d[2], 10) || null
        ).getTime();
        //#endif
        //#ifdef MP-WEIXIN
        var timestamp = new Date(dateTimeStamp).getTime()
        //#endif
        var minute = 1000 * 60;
        var hour = minute * 60;
        var day = hour * 24;
        var halfamonth = day * 15;
        var month = day * 30;
        var year = day * 365;
        var now = new Date().getTime();
        var diffValue = now - timestamp;
        var result;
        if (diffValue < 0) {
          return;
        }
        var yearC = diffValue / year;
        var monthC = diffValue / month;
        var weekC = diffValue / (7 * day);
        var dayC = diffValue / day;
        var hourC = diffValue / hour;
        var minC = diffValue / minute;
        if (yearC >= 1) {
          result = "" + parseInt(yearC) + "年前";
        } else if (monthC >= 1) {
          result = "" + parseInt(monthC) + "月前";
        } else if (weekC >= 1) {
          result = "" + parseInt(weekC) + "周前";
        } else if (dayC >= 1) {
          result = "" + parseInt(dayC) + "天前";
        } else if (hourC >= 1) {
          result = "" + parseInt(hourC) + "小时前";
        } else if (minC >= 1) {
          result = "" + parseInt(minC) + "分钟前";
        } else
          result = "刚刚";
        return result;
      }
    }
  }
</script>

<style scoped>
  .cu-list.menu-avatar>.cu-item .action {
    width: 200upx;
    text-align: center;
  }

  .blog-img {
    width: 130upx;
    height: 130upx;
    background-position: center;
    background-size: cover;
    background-repeat: no-repeat;
    float: right;
    margin-right: 20rpx;
  }
</style>