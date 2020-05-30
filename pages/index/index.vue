<template>
	<view class="container">
	  <view class="userinfo">
	    
	      
	      <text @click="initTheKeyPlug">初始化蓝牙插件(长按嘀嘀后再点)</text>
	     
	      <!-- <view class="addLockAndMan"> -->
	        <text @click="addMyLock">添加蓝牙锁(底部提示可以添加了再点)</text>
	        <text @click="delMyLock">删除(添加后再点)</text>
	      <!-- </view> -->
	
	      
	
	      <text @click="addRawInfo">获取初始信息配对--配对后再操作别的功能</text>
	      <text @click="addDNAInfo">获取DNA信息</text>
	      <text @click="openLock">开锁</text>
	      
	      <text @click="addPwdKey">添加钥匙密码138000</text>
		  
		  
	
	      <view  style="display: flex;flex-direction: column;" v-for="(item,index) in myKeys" :key="index">
		   <text>钥匙ID:{{item}}</text>
	        <text @click="deleteKey(item,index)">点击删除</text>
	        <text @click="enableKey(item)"> 钥匙使能</text>
	       <text @click="changeKeyPwd(item)">修改密码微133133</text>
	       <text @click="unableKey(item)">钥匙禁止</text>
	
	        
	
	
	      </view>
	
	
	      <text @click="getKeyList">同步钥匙列表</text>
	       <text @click="syncRecord">查看门锁记录</text>
	      <text @click="getSystemInfo">获取系统参数</text>
	      <text @click="getBroadCast">获取广播数据</text>
	      <text @click="getSystemInfo" >读取系统状态和参数</text>
	
	
	      <!-- <text @click="addHotelRoom" >设置酒店房间信息</text>
	      <text @click="getRoomInfo" >获取房间信息</text>
	      <text @click="setHotelInfo" >设置系统参数</text>
	      <text @click="getHotelSysInfo" >获取系统参数</text> -->
	
	
	
	
	      <text >--------------------------------</text>
	      <text >实时log输出</text>
	       <text >{{currentLog}}</text>
	   
	  </view>
	  
	</view>

</template>

<script>
	
	
	
	
	
	
	export default {
		data() {
			return {
				title: 'Hellosimda',
				bleDeviceList:[],
				 motto: 'Hello World',
				    userInfo: {},
				    hasUserInfo: false,
				    canIUse: wx.canIUse('button.open-type.getUserInfo'),
				    plugin:[],
				    myKeys:[],
				    currentLog:''
			}
		},
		
		onReady:function(){
		    console.info("---------onReady -->");
		 },
		
		
		  onLoad: function () {
		    
		  },
		methods: {
			
			  //重写setdata
			  setData:function(obj){    
			  let that = this;    
			  let keys = [];    
			  let val,data;    
			  Object.keys(obj).forEach(function(key){    
			   keys = key.split('.');    
			   val = obj[key];    
			   data = that.$data;    
			   keys.forEach(function(key2,index){    
			       if(index+1 == keys.length){    
			           that.$set(data,key2,val);    
			       }else{    
			           if(!data[key2]){    
			              that.$set(data,key2,{});    
			           }    
			       }    
			       data = data[key2];    
			   })    
			  });    
			  } ,
			  
			  
			
			  //初始化插件
			  initTheKeyPlug:function() {
			
			    console.info("---------myPlugin initTheKeyPlug -->");

			     var Plugin = requirePlugin("BLELockSDK");
			     this.plugin=new Plugin({
			       // keyGroupId: '903'
			     });
				 
				 
				 
				 
			     console.info("new Plugin", this.plugin);
			    //  this.setData({
			    //   currentLog:'正在初始化,请稍候'
			    // })
			 
			    var _this=this;
			 
			     this.plugin.on('ready',function(plugin){
			 
			       console.info("plugin is on ready", plugin);
			       _this.setData({
			        currentLog:'初始化成功,可以开始添加了'
			      })
			     });
			 
			     // 监听“开锁”事件上报
			     this.plugin.on("report:openLock", function(data) {
			       console.info("plugin is on opened lock, data is ", data);
			       this.setData({
			         currentLog:JSON.stringify(data)
			       })
			     });
			 
			     // 监听“运行错误”事件
			     this.plugin.on("error", function(err) {
			       console.info("plugin is on error -->", err);
			     });
			
			     // 监听“d断线”事件
			     
			     this.plugin.on("close",function(state){
			       console.log("断线了,重连");
			       setTimeout(() => {
			         //重连操作
			         _this.reconnect();
			       }, 3000);
			     });
			
			     //监听添加钥匙
			     this.plugin.on('addKey', res => {
			      console.info("addKeyaddKeyaddKeyaddKey", res);
			      if(res.errCode == '01'){
			        // 成功
			        wx.showToast({
			          title: '已添加钥匙'+res.data.lockKeyId,
			        })
					
					_this.myKeys.push(res.data.lockKeyId);
			        // var _this=this;
			      // var cus=_this.myKeys;
			      // cus.push(res.data.lockKeyId);
				  // _this.myKeys=cus;
				  _this.currentLog='已添加钥匙'+res.data.lockKeyId+'时间:9:15-23:15';
				  _this.$forceUpdate();
			      // _this.setData({
			      //   myKeys:cus,
			      //   currentLog:'已添加钥匙'+res.data.lockKeyId+'时间:9:15-23:15'
			      // })
			
			      }else{
			        // 其他原因
			      }
			    })
			 
			
			
			
			    
			  },
			
			  //添加蓝牙锁
			  addMyLock:function(){
			    this.plugin.addBluetoothDevice().then(res => { 
			      console.info(res);
			      wx.showToast({
			        title: '已添加',
			      })
			
			      //存储adminauthcode
			    wx.setStorageSync('adminAuthCode', res.data.DNAInfo.adminAuthCode);
			    wx.setStorageSync('aesKey', res.data.DNAInfo.aesKey);
			    wx.setStorageSync('lockMac', res.data.DNAInfo.lockMac);
			    console.log('存储adminAuthCode'+res.data.DNAInfo.adminAuthCode);
			
			    console.log(res.data.DNAInfo);
			
			    // 解析密钥
			    var s = "";
			    for (
			      var i = 0, aes = res.data.DNAInfo.aesKey, l = aes.length;
			      i < l;
			      i = i + 2
			    ) {
			      var a = aes[i] + aes[i + 1];
			
			      s += String.fromCharCode(`0x${a}`);
			    }
			
			    console.log("aesKey ------------>", s);
			
			      
			
			      var _this=this;
			
			      this.setData({
			        currentLog:'蓝牙锁已添加,3秒后自动配对'
			      })
			
			     setTimeout(() => {
			       _this.addRawInfo();
			     }, 3000);
			
			     }).catch(err => { 
			      wx.showToast({
			        title: '添加错误'+err,
			      })
			     })
			
			  },
			
			
			
			  reconnect:function(){
			    console.info("---------重连操作 -->");
			    let mylockMac = wx.getStorageSync('lockMac');
			    let myadminAuthCode = wx.getStorageSync('adminAuthCode');
			    let myaesKey = wx.getStorageSync('aesKey');
			    console.log(mylockMac+myadminAuthCode+myaesKey);
			
			     // 引入插件
			     var Plugin = requirePlugin("BLELockSDK");
			     this.plugin=new Plugin({
			       lockMac: mylockMac,
			       secret:myaesKey,
			       authCode:myadminAuthCode
			
			     });
			     console.info("new Plugin", this.plugin);
			     this.setData({
			      currentLog:'15秒后自动重新链接'
			    })
			
			     setTimeout(() => {
			      this.addMyLock();
			     }, 13000);
			  },
			
			
			  //修改钥匙密码
			  changeKeyPwd:function(value){
			    this.plugin.updatePassword({
			      newPassword: "133133",
			      lockKeyId: value.currentTarget.dataset.value,
			    }).then(res => {
			      this.setData({
			        currentLog:'钥匙密码已修改为133133'
			      })
			
			
			    }).catch(err => {
			      this.setData({
			        currentLog:'钥匙密码修改失败'
			      })
			
			
			    })
			
			
			  },
			
			
			  //同步钥匙列表
			  getKeyList:function(res){
			    this.plugin.synchronizeKeyList().then(res => { 
			
			      this.setData({
			        currentLog:'同步钥匙列表成功'
			      })
			      wx.navigateTo({
					  url:'./recordRavInfo?data='+JSON.stringify(res)

			      })
			
			    }).catch(err => { 
			      this.setData({
			        currentLog:'同步钥匙列表失败'
			      })
			
			
			     })
			
			  },
			
			
			
			  delMyLock:function(){
			    this.plugin.removeBluetoothDevice().then(res => { 
			      wx.showToast({
			        title: '已删除',
			      })
			      this.setData({
			        currentLog:'钥匙已删除'
			      })
			     }).catch(err => {
			      wx.showToast({
			        title: '删除错误'+err,
			      })
			     })
			  },
			
			  //获取初始信息
			  addRawInfo:function(){
			    console.log('获取初始信息');
			    this.plugin
			      .validateLockInitialInfo({
			        isSuccess: true
			      })
			      .then(function(res) {
			        console.log("validateLockInitialInfo res -->", res);
			      })
			      .catch(function(err) {
			        console.log("validateLockInitialInfo err -->", err);
			      });
			  },
			
			  //获取dna信息
			  addDNAInfo:function(){
			    console.log('获取dna信息');
			    this.plugin.readDNAInfo().then(res => {
			      console.log(res);
			      //存储lockMac
			    wx.setStorageSync('lockMac', res.data.lockMac);
			    console.log('存储lockmac'+res.data.lockMac);
			      wx.navigateTo({
			       url:'./recordRavInfo?data='+JSON.stringify(res)
			      })
			     }).catch(err => { 
			        wx.showToast({
			          title: '错误了',
			        })
			      })
			
			  },
			
			  //开锁
			  openLock:function(){
			    console.log('点击开锁');
			    this.plugin.openLock().then(res => { 
			      console.log(res); this.setData({
			        currentLog:'已开锁'
			      })
			     }).catch(err => {
			      console.log(err);
			      })
			
			  },
			
			
			  //添加钥匙
			  addPwdKey:function(){
			    // 有效日期
			      var validStartTime = new Date("2018-12-24");
			      validStartTime.setHours(9, 12, 15);//九点12分15秒开始有效
			      validStartTime = Math.floor(validStartTime.getTime() / 1000);
			
			      // 结束日期
			      var validEndTime = new Date("2020-12-25");
			      validEndTime.setHours(23, 12, 15);//23点12分15秒失效
			      validEndTime = Math.floor(validEndTime.getTime() / 1000);
			
			      var lockKeyId = 0;  // 钥匙ID由设备分配
			      var keyGroupId = 901;    // 用户组ID
			      var usageCount = 10;    // 使用次数
			
			
			      var options2 = {
			        lockKeyId,
			        keyGroupId,
			        validStartTime,
			        validEndTime,
			        usageCount,
			
			        type: 1,  // 按内容添加
			        keyType: 1,   // 密码类型
			        validTimeMode: 1,   // 0：有效期授权， 1：周期重复时间段授权
			        key: "138000",    // 内容值
			        dayStartTime: 10 * 60,    // 起始分钟数 
			        dayEndTime: 18 * 60,    // 结束分钟数
			        weeks: [0,1,2,3,4,5,6]    // 每周的周二、周四、周五
			      }
			
			      this.plugin.addKey(options2);
			     
			  },
			
			  // 按照钥匙 ID 进行使能
			  enableKey:function(value){
			    
			      const options = {
			        mode: 1,
			        lockKeyId: value,
			        value: true
			      }
			      this.plugin.enableKey(options).then(res => { 
			        console.log(res); this.setData({
			          currentLog:'钥匙已使能'
			        })
			
			      }).catch(err => { 
			        
			      })
			
			  },
			  // 按照钥匙 ID 进行禁止
			  unableKey:function(value){
			    
			    const options = {
			      mode: 1,
			      lockKeyId: value,
			      value: false
			    }
			    this.plugin.enableKey(options).then(res => { 
			      console.log(res); this.setData({
			        currentLog:'钥匙已禁止'
			      })
			
			    }).catch(err => { 
			      
			    })
			
			},
			
			  deleteKey:function(eee,theIndex){
			    console.log('删除这个钥匙'+ eee);
			    
			
			
			        // 按照内容删除：指定卡号或密码的卡片/密码钥匙  myPlugin 重连操作
			        var options3 = {
			          mode: 2,
			          keyType: [1],
			          key: '138000',
			        }
			
			        this.plugin.removeKey(options3).then(res => { 
			          wx.showToast({
			            title: '删除成功',
			          })
			          var _this=this;
			          var cus=_this.myKeys;
			          cus.splice(theIndex,1);
			          _this.setData({
			            myKeys:cus,
			            currentLog:'钥匙已删除'
			          })
			}).catch(err => { 
			
			        })
			
			  },
			  
			
			  syncRecord:function(){
			
			
			    this.plugin.synchronizeLockRecord({
			      offset: 0,
			      limit: 10
			    }).then(res => { 
			        wx.navigateTo({
			          url:'./recordRavInfo?data='+JSON.stringify(res)
			        })
			
			    }).catch(err => { 
			      wx.showToast({
			        title: '获取记录失败',
			      })
			     })
			  },
			
			  //获取信息
			  getSystemInfo:function(){
			    this.plugin.readSystemInfo().then(res => { 
			      wx.navigateTo({
			        url:'./recordRavInfo?data='+JSON.stringify(res)
			      })
			     }).catch(err => { 
			
			     })
			
			  },
			
			  //获取广播数据
			  getBroadCast:function(){
			    this.plugin.getBroadcastData().then(res => {
			      console.log("broadcast data: ", res);
			      wx.navigateTo({
			        url:'./recordRavInfo?data='+JSON.stringify(res)
			      })
			    }).catch(err => {
			      console.log("broadcast err: ", err);
			    })
			
			  },
			
			  //设置酒店房间信息（酒店用）addRawInfo
			  addHotelRoom:function(){
			    	const options = {
			      	  hotelId: 'ab12345a7821',
			      	  hotelBuildingId: 2,
			      	  hotelFloorId: 10,
			      	  hotelRoomId: 20,
			      	  hotelSlaveRoomId: 'A',
			      	  hotelAES128: '83okh0zBGyoJhYfW',
			      	  encryptionSector: 1
			      	}
			      	this.plugin.setRoomInfoForHotel(options).then(res => { 
			          this.setData({
			            currentLog:'设置酒店房间信息成功'
			          })
			
			        } ).catch(err => { 
			          console.log(err);
			          this.setData({
			            currentLog:'设置酒店房间信息失败'
			          })
			         })
			      
			  },
			
			  //设置酒店系统信息
			  setHotelInfo:function(){
			    const options = {
			      normallyOpenMode: 1,
			      	  volumeEnable: 2,
			      	  tongueChoose: 0,
			      	  backLockChoose: 1,
			      	  motorDirection: 1,
			      	  passwordEnable: 1,
			      	  tongueDetection: 0,
			      	  lockOpenTime: 6,
			      	  cardDetectiongSensitivity: 5,
			      	  passwordSensitivity: 5,
			         bleConnectInterval: 2,
			      	  setKeyFunction: 1,
			      	  lowPowerMotorDriverTimeAdd: 1,
			      
			      }
			      this.plugin.setSystemInfoForHotel(options).then(res => { 
			        this.setData({
			          currentLog:'设置酒店系统信息成功'
			        })
			        wx.navigateTo({
			          url:'./recordRavInfo?data='+JSON.stringify(res)
			        })
			
			      } ).catch(err => { 
			        console.log(err);
			        this.setData({
			          currentLog:'设置酒店系统信息失败'
			        })
			       })
			    
			},
			
			  getRoomInfo:function(){
			    this.plugin.getRoomInfoForHotel(options).then(res => { 
			      this.setData({
			        currentLog:'getRoomInfo成功'
			      })
			     } ).catch(err => { 
			      this.setData({
			        currentLog:'getRoomInfo失败'
			      })
			      })
			
			  },
			  getHotelSysInfo:function(){
			    this.plugin.getSystemInfoForHotel(options).then(res => { 
			      this.setData({
			        currentLog:'getSystemInfoForHotel成功'
			      })
			     } ).catch(err => { 
			      this.setData({
			        currentLog:'getSystemInfoForHotel失败'
			      })
			      })
			
			  },
			openBLE(){
				uni.openBluetoothAdapter({
				  success(res) {
				    console.log('openBluetoothAdapter'+res)
				  }
				})
				
				
				
				uni.startBluetoothDevicesDiscovery({
					success: (res) => {
						console.log('startBluetoothDevicesDiscovery'+res)
					}
				})
				
				var _this=this;
				
				uni.onBluetoothDeviceFound(function(eeeee){
					// console.log(eeeee)
					_this.bleDeviceList.splice(1,0,eeeee.devices[0]);
					console.log(_this.bleDeviceList)
				})
				
				
				
				// console.log('openBLEopenBLEopenBLEopenBLEopenBLEopenBLEopenBLE@@@@@@@');
				
				// 1.引用插件
				// const Plugin = requirePlugin('BLELockSDK');
				//   // 3.初始化插件
				//     const myPlugin = new Plugin(options)
				  
				//     // 4.监听ready事件
				//     myPlugin.on('ready', () => {
				  
				//       // 5.调用API
				//       // 插件所有的API都需要在插件响应ready事件之后才可以调用成功
				//       myPlugin
				//       .openLock()
				//       .then(res => {
				//         // 成功时调用
				//         // do something
						
				// 		console.log(res);
						
				//       })
				//       .catch(err => {
				// 		  console.log(err);
				//         // 失败时调用
				//         // debug
				//       })
				//     })
			}
			
			 
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
	
	
	.userinfo {
	  display: flex;
	  flex-direction: column;
	  align-items: center;
	  line-height: 70rpx;
	}
	
	.userinfo-avatar {
	  width: 128rpx;
	  height: 128rpx;
	  margin: 20rpx;
	  border-radius: 50%;
	}
	
	.userinfo-nickname {
	  color: #aaa;
	}
	
	.usermotto {
	  margin-top: 0px;
	}
	
	
	.addLockAndMan{
	  display: flex;
	  flex-direction: row;
	}
</style>
