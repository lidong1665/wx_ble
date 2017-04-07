# wx_ble
微信小程序 蓝牙实现

在前面已经写了两篇关于Android蓝牙和ios 蓝牙开发的文章，今天带来的是微信小程序蓝牙实现。
 
 - [Android蓝牙](https://github.com/lidong1665/Android-ble)
 - [ios蓝牙（Swift）](https://github.com/lidong1665/Swift-BLE)
 
有一段时间没有。没有写关于小程序的文章了。3月28日，微信的api又一次新的更新。期待已久的蓝牙api更新。就开始撸一番。
## 1.简述

 - 蓝牙适配器接口是基础库版本 1.1.0 开始支持。
 - iOS 微信客户端 6.5.6 版本开始支持，Android 客户端暂不支持
 - 蓝牙总共增加了18个api接口。


## 2.Api分类

 - 搜索类
 - 连接类
 - 通信类

## 3.API的具体使用
详细见官网：

[https://mp.weixin.qq.com/debug/wxadoc/dev/api/bluetooth.html#wxgetconnectedbluethoothdevicesobject](https://mp.weixin.qq.com/debug/wxadoc/dev/api/bluetooth.html#wxgetconnectedbluethoothdevicesobject)

## 4. 案例实现
#### 4.1 搜索蓝牙设备

```
/**
 * 搜索设备界面
 */
Page({
  data: {
    logs: [],
    list:[],
  },
   onLoad: function () {
    console.log('onLoad')
var that = this;
// const SDKVersion = wx.getSystemInfoSync().SDKVersion || '1.0.0'
// const [MAJOR, MINOR, PATCH] = SDKVersion.split('.').map(Number)
// console.log(SDKVersion);
// console.log(MAJOR);
// console.log(MINOR);
// console.log(PATCH);

// const canIUse = apiName => {
//   if (apiName === 'showModal.cancel') {
//     return MAJOR >= 1 && MINOR >= 1
//   }
//   return true
// }

// wx.showModal({
//   success: function(res) {
//     if (canIUse('showModal.cancel')) {
//       console.log(res.cancel)
//     }
//   }
// })
     //获取适配器
      wx.openBluetoothAdapter({
      success: function(res){
        // success
        console.log("-----success----------");
         console.log(res);
         //开始搜索
       wx.startBluetoothDevicesDiscovery({
  services: [],
  success: function(res){
    // success
     console.log("-----startBluetoothDevicesDiscovery--success----------");
     console.log(res);
  },
  fail: function(res) {
    // fail
     console.log(res);
  },
  complete: function(res) {
    // complete
     console.log(res);
  }
})


      },
      fail: function(res) {
         console.log("-----fail----------");
        // fail
         console.log(res);
      },
      complete: function(res) {
        // complete
         console.log("-----complete----------");
         console.log(res);
      }
    })

     wx.getBluetoothDevices({
       success: function(res){
         // success
         //{devices: Array[11], errMsg: "getBluetoothDevices:ok"}
         console.log("getBluetoothDevices");
         console.log(res);
          that.setData({
          list:res.devices
          });
          console.log(that.data.list);
       },
       fail: function(res) {
         // fail
       },
       complete: function(res) {
         // complete
       }
     })

  },
  onShow:function(){
 

  },
   //点击事件处理
  bindViewTap: function(e) {
     console.log(e.currentTarget.dataset.title);
     console.log(e.currentTarget.dataset.name);
     console.log(e.currentTarget.dataset.advertisData);
     
    var title =  e.currentTarget.dataset.title;
    var name = e.currentTarget.dataset.name;
     wx.redirectTo({
       url: '../conn/conn?deviceId='+title+'&name='+name,
       success: function(res){
         // success
       },
       fail: function(res) {
         // fail
       },
       complete: function(res) {
         // complete
       }
     })
  },
})

```
### 4.2连接 获取数据

```

/**
 * 连接设备。获取数据
 */
Page({
    data: {
        motto: 'Hello World',
        userInfo: {},
        deviceId: '',
        name: '',
        serviceId: '',
        services: [],
        cd20: '',
        cd01: '',
        cd02: '',
        cd03: '',
        cd04: '',
        characteristics20: null,
        characteristics01: null,
        characteristics02: null,
        characteristics03: null,
        characteristics04: null,
        result,

    },
    onLoad: function (opt) {
        var that = this;
        console.log("onLoad");
        console.log('deviceId=' + opt.deviceId);
        console.log('name=' + opt.name);
        that.setData({ deviceId: opt.deviceId });
        /**
         * 监听设备的连接状态
         */
        wx.onBLEConnectionStateChanged(function (res) {
            console.log(`device ${res.deviceId} state has changed, connected: ${res.connected}`)
        })
        /**
         * 连接设备
         */
        wx.createBLEConnection({
            deviceId: that.data.deviceId,
            success: function (res) {
                // success
                console.log(res);
                /**
                 * 连接成功，后开始获取设备的服务列表
                 */
                wx.getBLEDeviceServices({
                    // 这里的 deviceId 需要在上面的 getBluetoothDevices中获取
                    deviceId: that.data.deviceId,
                    success: function (res) {
                        console.log('device services:', res.services)
                        that.setData({ services: res.services });
                        console.log('device services:', that.data.services[1].uuid);
                        that.setData({ serviceId: that.data.services[1].uuid });
                        console.log('--------------------------------------');
                        console.log('device设备的id:', that.data.deviceId);
                        console.log('device设备的服务id:', that.data.serviceId);
                        /**
                         * 延迟3秒，根据服务获取特征 
                         */
                        setTimeout(function () {
                            wx.getBLEDeviceCharacteristics({
                                // 这里的 deviceId 需要在上面的 getBluetoothDevices
                                deviceId: that.data.deviceId,
                                // 这里的 serviceId 需要在上面的 getBLEDeviceServices 接口中获取
                                serviceId: that.data.serviceId,
                                success: function (res) {
                                    console.log('000000000000' + that.data.serviceId);
                                    console.log('device getBLEDeviceCharacteristics:', res.characteristics)
                                    for (var i = 0; i < 5; i++) {
                                        if (res.characteristics[i].uuid.indexOf("cd20") != -1) {
                                            that.setData({
                                                cd20: res.characteristics[i].uuid,
                                                characteristics20: res.characteristics[i]
                                            });
                                        }
                                        if (res.characteristics[i].uuid.indexOf("cd01") != -1) {
                                            that.setData({
                                                cd01: res.characteristics[i].uuid,
                                                characteristics01: res.characteristics[i]
                                            });
                                        }
                                        if (res.characteristics[i].uuid.indexOf("cd02") != -1) {
                                            that.setData({
                                                cd02: res.characteristics[i].uuid,
                                                characteristics02: res.characteristics[i]
                                            });
                                        } if (res.characteristics[i].uuid.indexOf("cd03") != -1) {
                                            that.setData({
                                                cd03: res.characteristics[i].uuid,
                                                characteristics03: res.characteristics[i]
                                            });
                                        }
                                        if (res.characteristics[i].uuid.indexOf("cd04") != -1) {
                                            that.setData({
                                                cd04: res.characteristics[i].uuid,
                                                characteristics04: res.characteristics[i]
                                            });
                                        }
                                    }
                                    console.log('cd01= ' + that.data.cd01 + 'cd02= ' + that.data.cd02 + 'cd03= ' + that.data.cd03 + 'cd04= ' + that.data.cd04 + 'cd20= ' + that.data.cd20);
                                    /**
                                     * 回调获取 设备发过来的数据
                                     */
                                    wx.onBLECharacteristicValueChange(function (characteristic) {
                                        console.log('characteristic value comed:', characteristic.value)
                                        //{value: ArrayBuffer, deviceId: "D8:00:D2:4F:24:17", serviceId: "ba11f08c-5f14-0b0d-1080-007cbe238851-0x600000460240", characteristicId: "0000cd04-0000-1000-8000-00805f9b34fb-0x60800069fb80"}
                                        /**
                                         * 监听cd04cd04中的结果
                                         */
                                        if (characteristic.characteristicId.indexOf("cd01") != -1) {
                                            const result = characteristic.value;
                                            const hex = that.buf2hex(result);
                                            console.log(hex);
                                        }
                                        if (characteristic.characteristicId.indexOf("cd04") != -1) {
                                            const result = characteristic.value;
                                            const hex = that.buf2hex(result);
                                            console.log(hex);
                                            that.setData({ result: hex });
                                        }

                                    })
                                    /**
                                     * 顺序开发设备特征notifiy
                                     */
                                    wx.notifyBLECharacteristicValueChanged({
                                        deviceId: that.data.deviceId,
                                        serviceId: that.data.serviceId,
                                        characteristicId: that.data.cd01,
                                        state: true,
                                        success: function (res) {
                                            // success
                                            console.log('notifyBLECharacteristicValueChanged success', res);
                                        },
                                        fail: function (res) {
                                            // fail
                                        },
                                        complete: function (res) {
                                            // complete
                                        }
                                    })
                                    wx.notifyBLECharacteristicValueChanged({
                                        deviceId: that.data.deviceId,
                                        serviceId: that.data.serviceId,
                                        characteristicId: that.data.cd02,
                                        state: true,
                                        success: function (res) {
                                            // success
                                            console.log('notifyBLECharacteristicValueChanged success', res);
                                        },
                                        fail: function (res) {
                                            // fail
                                        },
                                        complete: function (res) {
                                            // complete
                                        }
                                    })
                                    wx.notifyBLECharacteristicValueChanged({
                                        deviceId: that.data.deviceId,
                                        serviceId: that.data.serviceId,
                                        characteristicId: that.data.cd03,
                                        state: true,
                                        success: function (res) {
                                            // success
                                            console.log('notifyBLECharacteristicValueChanged success', res);
                                        },
                                        fail: function (res) {
                                            // fail
                                        },
                                        complete: function (res) {
                                            // complete
                                        }
                                    })

                                    wx.notifyBLECharacteristicValueChanged({
                                        // 启用 notify 功能
                                        // 这里的 deviceId 需要在上面的 getBluetoothDevices 或 onBluetoothDeviceFound 接口中获取
                                        deviceId: that.data.deviceId,
                                        serviceId: that.data.serviceId,
                                        characteristicId: that.data.cd04,
                                        state: true,
                                        success: function (res) {
                                            console.log('notifyBLECharacteristicValueChanged success', res)
                                        }
                                    })

                                }, fail: function (res) {
                                    console.log(res);
                                }
                            })
                        }
                            , 1500);
                    }
                })
            },
            fail: function (res) {
                // fail
            },
            complete: function (res) {
                // complete
            }
        })
    },

    /**
     * 发送 数据到设备中
     */
    bindViewTap: function () {
        var that = this;
        var hex = 'AA5504B10000B5'
        var typedArray = new Uint8Array(hex.match(/[\da-f]{2}/gi).map(function (h) {
            return parseInt(h, 16)
        }))
        console.log(typedArray)
        console.log([0xAA, 0x55, 0x04, 0xB1, 0x00, 0x00, 0xB5])
        var buffer1 = typedArray.buffer
        console.log(buffer1)
        wx.writeBLECharacteristicValue({
            deviceId: that.data.deviceId,
            serviceId: that.data.serviceId,
            characteristicId: that.data.cd20,
            value: buffer1,
            success: function (res) {
                // success
                console.log("success  指令发送成功");
                console.log(res);
            },
            fail: function (res) {
                // fail
                console.log(res);
            },
            complete: function (res) {
                // complete
            }
        })

    },
    /**
     * ArrayBuffer 转换为  Hex
     */
    buf2hex: function (buffer) { // buffer is an ArrayBuffer
        return Array.prototype.map.call(new Uint8Array(buffer), x => ('00' + x.toString(16)).slice(-2)).join('');
    }
})
```

## 5.效果展示
![这里写图片描述](http://img.blog.csdn.net/20170331135141121?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDA0NjkwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

发送校验指令。获取结果

![这里写图片描述](http://img.blog.csdn.net/20170331135155586?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDA0NjkwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
