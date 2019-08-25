<template>
	<view class="root">
		<view v-if="showSearch" class="base-padding mgb-30upx">
			<search />
		</view>
		
		<view class="base-padding base-margin-bottom">
			<swiper :style="'height:'+bannerHeight" :autoplay="autoplay" :indicator-dots="indicatorDots" :interval="interval" :duration="duration">
				<swiper-item v-for="banner in banners" :key="banner.id">
					<navigator :url="banner.link">
						<image :src="banner.image" class="radius-basic" :style="'height:'+bannerHeight+';width:'+bannerWidth"></image>
					</navigator>
				</swiper-item>
			</swiper>
		</view>
		
		<!--  推荐  -->
		<view v-if="recommendBooks.length>0" class='panel base-padding recommend base-margin-bottom'>
		  <view class='panel-heading'>
		    <view class='panel-title font-lv1 strong'>最新推荐</view>
		  </view>
		  <view class='panel-body'>
			<scroll-book :books="recommendBooks" :width="bannerWidth"></scroll-book>
		  </view>
		</view>
		
		<!--  各种分类的书籍的展示  -->
		<block v-for="category in categoryBooks" :key="category.id">
		  <view v-if="category.books" class='panel base-padding base-margin-bottom cate-data'>
			<view class='panel-heading'>
			  <view class='panel-title font-lv1 strong'>{{category.title}}
				<navigator :url="'/pages/list/list?tab=new&cid='+category.id" class='pull-right color-link font-lv3'>更多</navigator>
			  </view>
			</view>
			<view class='panel-body'>
			  <list-book :books="category.books" />
			</view>
		  </view>
		</block>		
		
	</view>
	
	
</template>

<script>
	import scrollBook from '../../components/scrollBook.vue'
	import search from '../../components/search.vue'
	import listBook from '../../components/listBook.vue'
	
	import api from '../../utils/api.js'
	import util from '../../utils/util.js'
	import config from '../../config.js'
	
	export default {
		data() {
			return {
				indicatorDots: true,
				autoplay: true,
				interval: 3000,
				duration: 500,
				bannerWidth: '100%',
				bannerHeight: 'auto',
				showSearch: false, // 内容完全加载完成之后再显示搜索框
				banners: [],
				categoryBooks: [],
				recommendBooks: []
			}
		},
		onLoad() {
			let info = util.getSysInfo()
			this.bannerWidth =  info.bannerWidth + "px",
			this.bannerHeight = info.bannerHeight + "px"
			if(config.debug) console.log(this.bannerWidth, this.bannerHeight)
			this.loadData()
		},
		methods: {
			loadData() {
				util.loading('玩命加载中...')
			    let that = this
			    let cids = []
			    let categories = []
			    api.getCategories().then(function(res) {
			      if (res.length > 0) {
			        categories = res.filter(function(category) {
			          let b = category.pid == 0 && category.cnt > 0
			          if (b) cids.push(category.id)
			          return b
			        })
			      }
			      if (config.debug) console.log(res, categories, cids)
			    }).catch(function(e) {
			      console.log("api.getCategories()", e)
			    }).finally(function() {
			      let banners = []
			      let recommendBooks = []
			      let bookLists = []
			      Promise.all([util.request(config.api.banners), util.request(config.api.bookLists, {
			        page: 1,
			        size: 12,
			        sort: 'latest-recommend'
			      }), util.request(config.api.bookListsByCids, {
			        page: 1,
			        size: 5,
			        sort: 'new',
			        cids: cids.join(',')
			      })]).then(function([resBanners, resRecommendBooks, resBookLists]) {
			        if (config.debug) console.log(cids, resBanners, resRecommendBooks, resBookLists)
			        if (resBanners.data && resBanners.data.banners) banners = resBanners.data.banners
			        if (resRecommendBooks.data && resRecommendBooks.data.books) recommendBooks = resRecommendBooks.data.books
			        if (resBookLists.data && resBookLists.data.books) {
			          categories = categories.map(function(category) {
			            let book = resBookLists.data.books[category.id]
			            if (book != undefined && book.length > 0) {
			              category.books = book
			            } else {
			              category.books = []
			            }
			            return category
			          })
			        }
			      }).catch(function(e) {
			        console.log(e)
			      }).finally(function() {
			        that.banners = banners
					that.categoryBooks = categories
					that.recommendBooks = recommendBooks
					that.showSearch= true
			        uni.hideLoading()
			      })
			    })
			  },
		},
		components:{
			scrollBook,
			search,
			listBook,
		}
	}
</script>

<style>
</style>