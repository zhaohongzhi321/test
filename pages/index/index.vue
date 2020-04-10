<template>
	<view>
		<u-nav-bar title="提现" leftIcon="back" @click-left="goBack" fixed="true" backgroundColor="#ffffff" color="#000000"></u-nav-bar>
		<view class="bgfff paddingt_30 paddingb_30 paddingl_30">
			<view class="fontSize_30 marginb_10">锡望卡  {{payerNo}}</view>
			<view class="fontSize_20 orange">可用余额：{{availBalance}}</view>
		</view>
		<view class="bgfff paddingb_20">
			<view class="paddingt_40 marginl_30 borderb">
				<text class="fontSize_60 bold">￥</text>
				<input step="0.01" class="input bold fontSize_48" maxlength="12" @input="oninput" placeholder-class="placeholder" v-model="amount" type="number" placeholder="请输入提现金额"/>
			</view>
			<view class="bigAmount"v-show="bigAmount">大写金额：{{bigAmount}}</view>
			<!-- <view class="unit fontSize_25" v-text="unit"></view> -->
		</view>
		<view class="payerNobox margint_20 fontSize_32 borderb" @click="showPayeeNoList">
			<text>提现至</text>
			<image class="arrowIcon" src="../../static/images/loan/arrow.png"></image>
			<!-- <text class="floatr marginr_60 fontSize_28" style="color: #BBBBBB;" v-show="!payeeBank">请选择</text> -->
			<text class="floatr marginr_60" v-show="payeeBank">{{payeeBank}}（{{payeeNo}}）</text>
			<image class="bankIcon floatr" v-show="payeeBankNo" :src="'../../static/images/bankIcon/'+payeeBankNo+'.png'"></image>
		</view>
		<u-cell title="用途" slot-right>
			<input class="fontSize_28 textAlignr width_650" v-model="purpose" placeholder="请输入提现用途" @click.stop=""/>
		</u-cell>
		<u-button style="margin: 50rpx;" width="650rpx" height="80rpx" shape="circle" @click="withdraw">确认提现</u-button>
		<u-actionsheet :show="showActionSheet" :tips="tips" :item-list="itemList" :mask-closable="maskClosable"
			:color="color" :size="size" :cancelColor="cancelColor" :is-cancel="isCancel"  @click="itemClick" @cancel="closeActionSheet">
		</u-actionsheet>
	</view>
</template>

<script>
	export default {
		// onLoad(data) {
		// 	const self = this;
		// 	this.$method.acctListQry({}, function(data){
		// 		var payerNolist = data.list
		// 		for(var i=0;i<payerNolist.length;i++){
		// 			if(payerNolist[i].acctClass == '2'){
		// 				self.payerBank = payerNolist[i].bankName;
		// 				self.payerNo = payerNolist[i].acctNoView;
		// 				self.acctNo = payerNolist[i].acctNo;
		// 				self.availBalance = payerNolist[i].amount;
		// 			}
		// 		}
		// 		self.payeeNoList = data.bindAcList
		// 		self.payeeBank = self.payeeNoList[0].settleBankName;					
		// 		self.payeeNo = self.payeeNoList[0].acctNoView;					
		// 		self.acctNo = self.payeeNoList[0].acctNo;					
		// 		self.payeeBankNo = self.payeeNoList[0].settleBranch;
		// 	});
		// },
		data() {
			return {
				payerBank:'',
				payerNo:'',
				acctNo:'',
				availBalance:'',
				amount:'',
				unit:'',
				payeeNoList:[],
				payeeBank:'',
				payeeNo:'',
				payeeBankNo:'',
				purpose:'',
				
				verificationCode:'',
				trsPassWord:'',
				trsPassWordPin:'',
				serialNo:'',
				
				showActionSheet: false,
				maskClosable: true,
				tips: "",
				item:'',
				itemList: [],
				color: "#9a9a9a",
				size: 26,
				isCancel: true,
				cancelColor:"",
			}
		},
		methods: {
			withdraw(){
				if(this.$regular.isEmpty(this.amount,'请输入提现金额')) return
				if(this.$regular.isEmpty(this.payeeNo,'请选择收款账户')) return
				if(parseFloat(this.amount) <= 0){
					uni.showModal({
						title:'提示',
						content:'提现金额必须大于零',
						showCancel:false
					})
					return
				}
				if(parseFloat(this.amount)>parseFloat(this.availBalance)){
					uni.showModal({
						title:'提示',
						content:'提现金额不得大于可用余额',
						showCancel:false
					})
					return
				}
				const self = this;
				var params = {
					acctNo:self.acctNo,
					agreementType:'2',
				}
				//银行卡签约状态查询
				// self.$http.post('pweb-query.QryAccountSignStatus.do',params,{
				// 	success: function(res) {
						self.$method.getWeexPswTwo({
							title:'提现',//弹窗标题
							acctNo:self.acctNo,
							needSms:'0'
						},function(data){
							self.verificationCode = data.uniTransPswOneSmsValue;
							self.trsPassWord = data.uniTransPswOnePsw;
							self.trsPassWordPin = data.uniTransPswOnePswPin;
							self.serialNo = data.serialNo;
							var params = {
								// cardNo:self.acctNo,
								// acctNo:self.acctNo,
								accountNo:self.acctNo,
								amount:self.amount,
							}
							//提现确认
							self.$http.post('pweb-transfer.WithdrawComfirm.do',params,{
								success: function(res) {
									var _tokenName = res._tokenName; 
									var params = {
										accountNo:self.acctNo,
										amount:self.amount,
										verificationCode:self.verificationCode,
										trsPassWord:self.trsPassWord,
										trsPassWordPin:self.trsPassWordPin,
										_tokenName:_tokenName,
										serialNo:self.serialNo
									}
									//提现
									self.$http.post('pweb-transfer.Withdraw.do',params,{
										success: function(res) {
											uni.navigateTo({
												url:'hopeCardWithdrawResult'
											})
											uni.setStorage({
												key:'withdrawInfo',
												data:{
													payerBank:self.payerBank,
													payerNo:self.payerNo,
													payeeBank:self.payeeBank,
													payeeNo:self.payeeNo,
													amount:self.amount,
													purpose:self.purpose,
												}
											})
										}
									});
								}
							});
						})
				// 	}
				// });
			},
			goBack(){
				context.finish();
			},
			oninput(e) {
				console.log(e);
				// var moneyMaxLeng = 0;
				
				// let val = e.target.value;
				// val = val.replace(/[^\d.]/g, ""); //清除"数字"和"."以外的字符
				// val = val.replace(/\.{2,}/g, "."); //只保留第一个. 清除多余的
				// val = val.replace(/^0+\./g, '0.');
				// val = val.match(/^0+[1-9]+/) ? val = val.replace(/^0+/g, '') : val
				// val = (val.match(/^\d*(\.?\d{0,2})/g)[0]) || ''
				// if (val.includes(".")) {
				//   let numDian = val.toString().split(".")[1].length;
				//   if (numDian === 2) {
				// 	moneyMaxLeng = val.length;
				//   }
				// } else {
				//   moneyMaxLeng = 8;
				// }
				// this.amount = val;
				// console.log(moneyMaxLeng);
				// console.log(this.amount );
				// let index = this.payNumList.findIndex((n) => n == val)
				// if (index != -1) {
				//   this.selectPayEq = index
				// } else {
				//   this.selectPayEq = '8888'
				// }
				
				
			    // 通过正则过滤小数点后两位
				// var self = this;
				// console.log(e);
				// self.$nextTick(() => {
				// 	self.amount = (e.target.value.match(/^\d*(\.?\d{0,2})/g)[0]) || null
				// })
				
			},
			showPayeeNoList(){
				this.item = "payeeNo";
				this.itemList = JSON.parse(JSON.stringify(this.payeeNoList));
				this.itemList.push({text: "添加银行卡(+)"});
				this.showActionSheet = true;
			},
			closeActionSheet: function() {
				this.showActionSheet = false
			},
			itemClick: function(e) {
				if(this.item == "payeeNo"){
					if(this.itemList.length-1 == parseInt(e.index)){
						uni.navigateTo({
							url:'elements'
						})
						this.closeActionSheet();
						return
					}
					this.payeeBank = this.itemList[parseInt(e.index)].settleBankName;
					this.payeeNo = this.itemList[parseInt(e.index)].acctNoView;
					this.acctNo = this.itemList[parseInt(e.index)].acctNo;
					this.payeeBankNo = this.itemList[parseInt(e.index)].settleBranch;
					for(var i=0;i<this.payeeNoList.length;i++){
						this.payeeNoList[i].color = "#666666";
					}
					this.payeeNoList[parseInt(e.index)].color = "#3C7FF7";
				}
				this.closeActionSheet();
			},
		},
		computed:{
			bigAmount:function(){
				return ''
			}
		},
	}
</script>

<style>
.input{
	margin-left: 20rpx;
	display: inline-block;
	width: 550rpx;
	position: relative;
	top: 10rpx;
}
.placeholder{
	color: #999999;
	font-weight: normal;
	font-size: 30rpx;
}
.bigAmount{
	font-size: 20rpx;
	color: #999999;
	padding: 10rpx 30rpx;
}
/* .unit{
	position: absolute;
	top: 240rpx;
	left: 136rpx;
	padding-left: 5rpx;
	color: #888888;
	border-left: 2rpx solid #888888;
} */
.arrowIcon{
	width: 60rpx;
	height: 60rpx;
	margin-right: 30rpx;
	position: absolute;
	right: 0rpx;
	top: 20rpx;
}
.bankIcon{
	width: 40rpx;
	height: 40rpx;
	margin-right: 30rpx;
	margin-top: 10rpx;
	/* position: absolute;
	left: 200rpx; */
}
.payerNobox{
	padding: 20rpx 30rpx;
	background: #ffffff;
	height: 60rpx;
	line-height: 60rpx;
	position: relative;
}
</style>
