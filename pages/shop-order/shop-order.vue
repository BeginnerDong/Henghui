<template>
	<view class="px-3 py-2">
		
		<view class="bg-white line-h p-r pb-1 mb-2 radius10 overflow-h font-24">
			<view class="py-3 px-2 line-h d-flex a-center j-sb p-r" @click="Router.navigateTo({route:{path:'/pages/shop-address/shop-address'}})">
				<view v-if="addressData.name">
					<view class="pb-3 font-30">{{addressData.name}}<text class="color9 pl-2 font-24">{{addressData.phone}}</text></view>
					<view>{{addressData.city+addressData.detail}}</view>
				</view>
				<view v-else>请添加收货地址</view>
				<image src="../../static/images/mall-icon3.png" class="R-icon"></image>
			</view>
			<image src="../../static/images/my order-icon1.png" class="order-icon p-a bottom-0 left-0 right-0"></image>
		</view>
		
		
		<view >
			<view class="bg-white px-2 radius10">
				<view class="" v-for="(item,index) in mainData" :key="index">
					<view class="d-flex a-center j-sb py-3">
						<image :src="item.product&&item.product.mainImg&&item.product.mainImg[0]?item.product.mainImg[0].url:''" class="shopImg"></image>
						<view class="shopCon d-flex flex-column flex-1 pl-2">
							<view class="tit avoidOverflow2 pb-2 flex-1">{{item.product&&item.product.title?item.product.title:''}}</view>
							<view class="font-22 color6 line-h pb-1">
								<span v-for="(c_item,key,c_index) in item.product.sku[item.skuIndex].label_description" :key="key">{{key}}:{{c_item}}</span>
							</view>
							<view class="colorR">
								<text class="price1 font-w pb-2 font-32">{{item.product&&item.product.sku&&item.product.sku[item.skuIndex]?item.product.sku[item.skuIndex].o_price:''}}</text>
								<text class="priceV font-w font-28"  style="color: #000000;margin-left: 10rpx;">{{item.product&&item.product.sku&&item.product.sku[item.skuIndex]?item.product.sku[item.skuIndex].price:''}}</text>
							</view>
						</view>
						
					</view>
					<view class="ggPart pb-1 flex1 px-2 bB-f5">
						<view class="py-3">购买数量</view>
						<view class="d-flex a-center count">
							<image src="../../static/images/shopping-icon2.png" class="count-icon1" @click="counter(index,'-')"></image>
							<view class="num text-center f5bj">{{item.count}}</view>
							<image src="../../static/images/shopping-icon3.png" class="count-icon2" @click="counter(index,'+')"></image>
						</view>
					</view>
				</view>
				
				<view class="flex py-3">
					<view class="font-26 pr-3">订单备注</view>
					<input type="text" v-model="passage2" placeholder="请输入" />
				</view>
			</view>
			
		</view>
		
		
		<view class="bg-white p-f left-0 right-0 d-flex carBot">
			<view class="font-22 d-flex a-center flex-1 px-3">总计： <text class="price1 font-30 font-w">{{totalPrice}}</text></view>
			<view class="carBtn" @click="submit">提交订单</view>
		</view>
		
		
		
		<!-- 提示 -->
		<view class="bg-mask" v-show="is_show">
			<view class="bg-white p-a radius10 text-center tc">
				<view class="txt">
					<view class="content ql-editor text-center" style="padding:0;" v-html="orderTipData.content">
					</view>
				</view>
				<view class="pb-2">
					<view  @click="phoneCall1(item)" style="margin-top: 10rpx;" v-for="(item,index) in phoneArray" :key="index">
						客服电话:{{item}}
					</view>
				</view>
				<view class="flex1 bT-f5 btn">
					<view class="w-50 bR-f5" @click="Router.redirectTo({route:{path:'/pages/shop-userOrder/shop-userOrder'}})">我知道了</view>
					<view class="colorM w-50" @click="phoneCall()">联系客服</view>
				</view>
			</view>
		</view>
		
		<view class="bg-mask" v-show="is_show1">
			<view class="bg-white p-a radius10 text-center tc">
				<view class="txt">
					<view class="content ql-editor text-center" style="padding:0;" v-html="tipsData.content">
					</view>
				</view>
				
				<view class="" style="margin-bottom: 30rpx;color: #888;"  @click="phoneCall1(item)" v-for="(item,index) in phoneArray" :key="index">联系客服：{{item}}</view>
				<view class="flex1 bT-f5 btn">
					<view class="w-50 bR-f5" @click="showToast1">我知道了</view>
					<view class="colorM w-50" @click="phoneCall()">联系客服</view>
				</view>
			</view>
		</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				Router:this.$Router,
				is_show:false,
				mainData:{},
				totle:0,
				addressData:{},
				is_show1:false,
				passage2:'',
				kefuData:{},
				tipsData: {},
				totalPrice:0,
				orderTipData:{},
				phoneArray:[]
			}
		},
		
		
		
		onLoad(options) {
			const self = this;
			uni.removeStorageSync('choosedAddressData');
			self.mainData = uni.getStorageSync('payPro');
			self.getKefuData();
			self.getUserInfoData()
			self.getTipsData();
			self.getOrderTipData();
			
		},
		
		onShow() {
			const self = this;
			if(uni.getStorageSync('choosedAddressData')){
				self.addressData = uni.getStorageSync('choosedAddressData')
				
			}else{
				self.getAddressData()
			};
		},
		
		methods: {
			
			phoneCall(){
				const self = this;
				uni.makePhoneCall({
					phoneNumber:self.phoneArray[0]
				})
			},
			
			phoneCall1(item){
				const self = this;
				uni.makePhoneCall({
					phoneNumber:item
				})
			},
			
			getOrderTipData() {
				const self = this;
				const postData = {};
				postData.searchItem = {
					thirdapp_id: 2,
					title: '订单提交提示',
					menu_id: 10
				};
				var callback = function(res) {
					if (res.info.data.length > 0) {
						self.orderTipData = res.info.data[0]
					}
					self.$Utils.finishFunc('getOrderTipData');
				}
				self.$apis.articleGet(postData, callback);
			},
			
			getTipsData() {
				const self = this;
				const postData = {};
				postData.searchItem = {
					thirdapp_id: 2,
					title: '普通会员提示（商城）',
					menu_id: 10
				};
				var callback = function(res) {
					if (res.info.data.length > 0) {
						self.tipsData = res.info.data[0]
					}
					console.log('menu', self.tipsData)
					self.$Utils.finishFunc('getTipsData');
				}
				self.$apis.articleGet(postData, callback);
			},
			
			getKefuData() {
				const self = this;
				const postData = {};
				// postData.tokenFuncName = 'getProjectToken';
				postData.searchItem = {
					menu_id: 3,
					thirdapp_id: 2
				};
				var callback = function(res){
					if(res.info.data.length > 0){
						self.kefuData = res.info.data[0];
					}
				}
				self.$apis.articleGet(postData, callback);
			},
			
			submit(){
				const self = this;
				if(self.userInfoData.behavior==1||self.userInfoData.user_type==1){
					self.addOrder()
				}else{
					self.is_show1 = true
				}
			},
			
			
			addOrder() {
				const self = this;	
				uni.setStorageSync('canClick', false);
				if(JSON.stringify(self.addressData) == '{}'){
					uni.setStorageSync('canClick', true);
					self.$Utils.showToast('请选择收货地址','none')
					return
				};
				var orderList = []
				var data = {};
				for (var i = 0; i < self.mainData.length; i++) {
					orderList.push({sku_id:self.mainData[i].sku_id,count:self.mainData[i].count,data: data,
					snap_address: self.addressData})
				}
				const postData = {}; 
				postData.orderList = self.$Utils.cloneForm(orderList);
				postData.data = {
					level:1,
					//pay_status:1,
					snap_address:self.addressData,
					passage2:self.passage2
				};
				postData.type = 1;
				postData.parent = 1;
				postData.tokenFuncName = 'getUserToken';
				const callback = (res) => {
					uni.setStorageSync('canClick', true);
					if (res && res.solely_code == 100000) {
						self.orderId = res.info.id;
						var array = self.$Utils.getStorageArray('cartData');
						for (var i = 0; i < orderList.length; i++) {
							for (var j = 0; j < array.length; j++) {
								if(orderList[i].sku_id == array[j].sku[array[j].skuIndex].id){
									self.$Utils.delStorageArray('cartData', orderList[i], 'id');
								}
							}
						};
						self.is_show = true;
					} else {		
						uni.showToast({
							title: res.msg,
							duration: 2000
						});
					};		
				};
				self.$apis.addOrder(postData, callback);
			},
			
			showToast1(){
				const self = this;
				self.is_show1 = false
			},
			
			getUserInfoData() {
				const self = this;
				const postData = {};
				postData.tokenFuncName = 'getUserToken';
				postData.getAfter = {
					staff:{
						tableName:'User',
						middleKey:'staff_no',
						key:'staff_no',
						searchItem:{
							status:1,
							user_type:1
						},
						condition:'='
					}
				};
				var callback = function(res){
					if(res.info.data.length > 0){
						self.userInfoData = res.info.data[0].info;
						if(res.info.data[0].staff&&res.info.data[0].staff[0]&&res.info.data[0].staff[0].info&&res.info.data[0].staff[0].info.phone!=''){
							self.phoneArray.push(res.info.data[0].staff[0].info.phone)
						}else{
							self.phoneArray.push(uni.getStorageSync('user_info').thirdApp.phone) 
						}
						self.countTotalPrice()
					}
					self.$Utils.finishFunc('getUserData');
				}
				self.$apis.userGet(postData, callback);
			},
			
			
			getAddressData() {
				const self = this;		
				const postData = {};
				postData.tokenFuncName = 'getUserToken';
				postData.searchItem = {
					isdefault:1
				};
				const callback = (res) => {
					if (res.info.data.length > 0) {
						self.addressData = res.info.data[0];
					}
				};
				self.$apis.addressGet(postData, callback);
			},
			
			showToast(){
				const self = this;
				self.is_show = !self.is_show
			},
			
			counter(index, type) {
				const self = this;
				if (type == '+') {
					self.mainData[index].count++;
				} else {
					if (self.mainData[index].count > 1) {
						self.mainData[index].count--;
					}
				};
				self.countTotalPrice();
			},
			
			countTotalPrice() {
				const self = this;
				self.totalPrice = 0;
				/* self.couponTotalPrice = self.$Utils.addItemInArray(self.pay.coupon, 'price'); */
				for (var i = 0; i < self.mainData.length; i++) {
					if(self.userInfoData.behavior==0){
						self.totalPrice += self.mainData[i].product.sku[self.mainData[i].skuIndex].o_price*self.mainData[i].count
					}else{
						self.totalPrice += self.mainData[i].product.sku[self.mainData[i].skuIndex].price*self.mainData[i].count
					}
					
				}
				self.totalPrice = parseFloat(self.totalPrice).toFixed(2)
				console.log(self.pay)
			},
			
		}
	}
</script>
<style>
html,body{height: 100%!important;background-color: #f5f5f5;}
page{background-color: #f5f5f5;}
</style>
<style scoped>
.order-icon{width: 100%;height: 4rpx;}
input{font-size: 24rpx;flex: 1;}
.num{background-color: #fff;}
.carBot{bottom: 0;}
.tc{width: 620rpx;margin: auto;left: 0;right: 0;top: 30%;}
.tc .txt{padding: 60rpx;}
.tc .btn{line-height: 100rpx;}
</style>
