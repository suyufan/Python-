# 云开发 quickstart

这是云开发的快速启动指引，其中演示了如何上手使用云开发的三大基础能力：

- 数据库：一个既可在小程序前端操作，也能在云函数中读写的 JSON 文档型数据库
- 文件存储：在小程序前端直接上传/下载云端文件，在云开发控制台可视化管理
- 云函数：在云端运行的代码，微信私有协议天然鉴权，开发者只需编写业务逻辑代码

## 参考文档

- [云开发文档](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/basis/getting-started.html)

----------



> 小程序父页面向子页面传值：url的方式
>
> 子页面给父页面传值：
>
>  - 子页面data-xx 绑定事件  事件中通过e.currentTarget.dataset.xx获得值 & getCurrentPages()[pages.length-2]获得父页面 父页面.接收值事件(值) setData
>
>    [视频讲解链接](https://www.bilibili.com/video/BV1tL4y1i7u4/?p=43&spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=d0a2e50aa23ef3b6624c497c2378249b)

#### 1.发布流程的问题

- 方式一

  ```
  1. 打开图片进行本地预览
  2. 输入文字 & 选择相应的信息
  3. 点击发布按钮
  	3.1 将本地图片上传到 腾讯云对象存储中COS（oss），并将COS中的图片地址返回。
  	3.2 将COS中的图片URL和文字等信息一起提交到后台。
  BUG：
  	在3.2步骤时可能拿不到COS中的图片。
  ```

  ```js
  function onClickSubmit(){
      // 耗时1分钟，不会阻塞。异步
      wx.request({
          url:"...",
          success:function(res){
              console.log(res)
          }
      })
      console.log(123);
  }
  // 可能出现 先打印123 之后请求发送完了才打印res
  ```

- 方式二（推荐）

  ```
  1. 打开图片进行本地预览
  2. 将本地图片上传到 腾讯云对象存储中COS
  3. 输入文字 & 选择相应的信息
  4. 发布：
  	必须上传完毕之后，才允许点击发布按钮。
  ```

  

### 2.闭包问题

```js
for (var i in ["alex", "changxin", "cck"]) {
    wx.request({
      url: 'xxxxx',
      success: function (res) {
        console.log(i);
      }
    })
}
// 打印结果：2 2 2
```

会输出三个2，而不是123

因为wx.request()是异步方法，三个wx.request()同时执行，而不是一个执行完了依次执行下一个。

怎么让输出结果为"alex", "changxin", "cck"呢？

// 闭包

```js
var dataList = ["alex", "changxin", "cck"]
for (var i in dataList) {
    
  function v (data) {
    wx.request({
      url: 'xxxxx',
      success: function (res) {
        console.log(data);
      }
    })
  }
    
 v(dataList[i])
}
```

或者 写成闭包的自执行函数形式

```js
var dataList = ["alex", "changxin", "cck"]
for (var i in dataList) {
  (function(data){
    wx.request({
      url: 'xxxxx',
      success: function (res) {
        console.log(data);
      }
    })
  })(dataList[i])
}
```



### 3.小红书滑到底部 自动刷新加载新帖子机制

设置minID和maxID（假如minID为27，maxID为100。minID对应滑到底部进行刷新，maxID对应顶部下拉刷新。）

下面以滑到底部刷新为例：（注意下拉顶部刷新的时候 加载完数据需要wx.stopPullDownRefresh()停止刷新）

在onReachBottom页面上拉触底事件处理函数中，将当前的minID: 27传给后端

后端过滤比27小的10数据，即26，25，，，17 返回给前端

```python
def get(self,request,*args,**kwargs):
    min_id = request.query_params.get('min_id')
    max_id = request.query_params.get('max_id')
    if min_id:
        queryset = models.News.objects.filter(id__lt=min_id).order_by('-id')[0:10] #倒序 26-17
    elif max_id:
        queryset = models.News.objects.filter(id__gt=min_id).order_by('id')[0:10] # 顺序 101-110
    else:
        queryset = models.News.objects.all().order_by('-id')[0:10]
    ser = NewsModelSerializer(instance=queryset,many=True)
    return Response(ser.data,status=200)
```



前端此时将17作为minID 消息列表newList: this.data.newList.concat(res.data)

> 优化

加一个判断，res.data.length如果为0，说明没有返回新数据，数据库中的数据都被显示了，此时

```js
wx.showToast({
    title: '已到底部',
    icon: 'none'
})
```



### 4.瀑布流布局

- 方式一：自己构造

    html文件，一个view中两个image

    ```html
    <view class='container'>
      <view class="item">
        <image src="https://hbimg.huabanimg.com/762eee0f99f9fbbc458fb70b0b86d0f8090ba45e7fb75-z1bDC7_fw236" mode="widthFix" ></image>
        <image src="https://hbimg.huabanimg.com/762eee0f99f9fbbc458fb70b0b86d0f8090ba45e7fb75-z1bDC7_fw236" mode="widthFix" ></image>
      </view>
      <view class="item">
        <image src="https://hbimg.huabanimg.com/1143ded46f1808fd460de68bb81d1513d7578d88543aa-cvwFGk_fw236" mode="widthFix" ></image>
        <image src="https://hbimg.huabanimg.com/762eee0f99f9fbbc458fb70b0b86d0f8090ba45e7fb75-z1bDC7_fw236" mode="widthFix" ></image>
      </view>
    </view>
    ```
    
    对应的wxss
    
    ```css
    .container{
      display: flex;
      flex-direction: row;
    }
    
    .container .item{
      width: 50%;
      overflow: hidden;
    }
    
    .container .item image{
      width: 100%;
    }
    ```
    
- 方式二：高级一点的css

    html文件，一个view中一个image，自动成两列分开布局

    ```html
    <view class="container">
      <view class="item">
        <image src="https://hbimg.huabanimg.com/1143ded46f1808fd460de68bb81d1513d7578d88543aa-cvwFGk_fw236" mode="widthFix" ></image>
      </view>
      <view class="item">
        <image src="https://hbimg.huabanimg.com/762eee0f99f9fbbc458fb70b0b86d0f8090ba45e7fb75-z1bDC7_fw236" mode="widthFix" ></image>
      </view>
      <view class="item">
        <image src="https://hbimg.huabanimg.com/762eee0f99f9fbbc458fb70b0b86d0f8090ba45e7fb75-z1bDC7_fw236" mode="widthFix" ></image>
      </view>
      <view class="item">
        <image src="https://hbimg.huabanimg.com/762eee0f99f9fbbc458fb70b0b86d0f8090ba45e7fb75-z1bDC7_fw236" mode="widthFix" ></image>
      </view>
    </view>
    ```

    对应的wxss

    ```css
    .container
    {
        -moz-column-count:2; /* Firefox */
        -webkit-column-count:2; /* Safari and Chrome */
        column-count:2;
    
        -moz-column-gap:20rpx; /* Firefox */
        -webkit-column-gap:20rpx; /* Safari and Chrome */
        column-gap:20rpx;
    }
    
    .container .item{
      break-inside: avoid-column;
      -webkit-column-break-inside: avoid; /* Safari and Chrome */
    }
    ```

    

### 5.发送请求

`http://localhost/api/comment/?root=12`

```js
wx.request({
    url: '`http://localhost/api/comment/',
    data: {
        root: rootId
    },
    method: 'GET'
})
```

