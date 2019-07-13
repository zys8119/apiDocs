# huoliaoWallet
火聊【红包、钱包】分支

# app交互

#### 第一步

>app需要给H5注册相关交互函数

```typescript
interface Function {
  //获取用户信息
  getUserInfo():userInfo;
  //关闭webViews
  closeWindow(resData:closeWindow):void;
  //转账异步通知
  TransferResult?(TransferData:TransferData):void;
  /**
  * @发送红包异步通知
  * @param {object} RedPackData 发送红包的数据
  */
  sendRedPacket?(RedPackData:RedPackData):void;
  /**
  * @拆红包异步通知
  * @param {string} h5RouterName h5下一步需要跳转的路由名称，
  * 注意：该方法按需调用，请参考第三步
  */
  openRedPack?(h5RouterName):void;
  /**
  * @联系客服
  */
  openCustomerService?():void;
}

interface TransferData {
    //转账金额
    amount:string|number;
    //对方账号id
    friendId: string;
    //本次转账的支付凭证
    payProve: number;
    //自己的账号id
    userId: number;
    //转账的备注，默认为null
    remarks: string;
}

interface RedPackData{
    //发红包数据
    data:RedPackData_Data;
    //发送红包的返回数据
    result:RedPackData_Result;
}

interface RedPackData_Data{
    //群id，当发送的为个人红包群id 默认为 0
    gid:string|number;
    //红包金额
    money:string|number;
    //红包备注
    msg:string;
    //红包个数
    num?:string|number;
    //红包类型，1：个人，2：群组
    type:number;
}

interface RedPackData_Result{
    code:number;
    result:RedPackData_Result_Result
}

interface RedPackData_Result_Result{
    id:number;
}

interface userInfo {
  userId:string|number;
  token:string;
  self?:boolean;
  gid?:string|number;
  pid?:string|number;
  type?:string;
  other?:Other;
  groupSize?:number;
}

interface Other {
  avatar?:string;
  nickName?:string;
}

interface closeWindow {
  //投诉数据：
    //{type:'1',title:"发布了不适合内容对我造成了骚扰"},
    //{type:'2',title:"存在欺诈骗钱行为"},
    //{type:'3',title:"此账号可能被盗用了"},
    //{type:'4',title:"存在侵权行为"},
    //{type:'5',title:"发布仿冒品信息"},
    //{type:'6',title:"冒充他人"},
  title:string;
  type:string|number;
}
```

#### 第二步

>H5通过app暴露的getUserInfo获取以下userInfo信息

```javascript
var userInfo = {
    //以下用于钱包模块
    //【必传】，用户的loginId
    userId:196,
    //【必传】，用户的token
    token:"AE+9LXQgZOgeMryMA/E7JZorxz/ZyzUnq6wKUHKXsZejjiJmZJCstZLTzmXfKoO0YsGZFer/eJd1J/tXAyl8YQ==",
    
    
    //以下用户红包模块
    //【必传】，用户的token
    token:"AE+9LXQgZOgeMryMA/E7JZorxz/ZyzUnq6wKUHKXsZejjiJmZJCstZLTzmXfKoO0YsGZFer/eJd1J/tXAyl8YQ==",
    //【非必传】，打开自己发送的个人红包时需要，判断是否是自己的个人红包，自己为true，不是不用传
    self:true,
    //【非必传】，发送群红包时需要，群id
    gid:"45",
    //【非必传】，除发红包以外时需要，红包id
    pid:2473,
    //【非必传】，拆红包时需要，判断是否是个人红包
    type:'1',
    //【非必传】，对方信息，个人红包详情时需要，但是建议必传
    other:{
        //必传,头像
        avatar: "http://qiniu.egretloan.com/FpB-7b1ZKnOXeN3WgIkgJaqTeCdl" ,
        //必传，昵称
        nickName: "北路",
        
        //以下可不传
        logId: "4641",
        monney: 0.07,
        time: "2019-06-02 14:06:04",
        userId: "1214",
        userName: "17858938486",
    },
    //【非必传】，群人数，发送群红包时需要，默认0个人
    groupSize:0,
}
```

#### 第三步

>h5路由说明及备注

```text
 1、钱包个人转账的时候app需要跳转到 【"/TransferAccount?userName=123456789&from=user"】,
    其中userName为用户的登录账号或手机号,
 2、红包发送时app需要跳转到 【"/HongbaoSend?group=true"】，其中group非必传，当发送群红包时，group参数设置成true
 3、拆红包的openRedPack异步通知是按需调动的.
    app如果想要使用该回调，那么在跳转拆红包页面时路由应该添加Notice参数，并设置为true 
    例如：【"/HongbaoOpen?Notice=true"】  
 4、投诉路由需要跳转到【"/Complaint"】,如果群投诉可以传参数【group=true】，例如【"/Complaint?group=true"】
 5、协议路由需要跳转到【"/Agreement"】,
```
>ios交互参考文档 https://github.com/marcuswestin/WebViewJavascriptBridge/blob/master/README.md

