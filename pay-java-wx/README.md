

## 微信支付简单例子

#### 支付配置

```java

              WxPayConfigStorage wxPayConfigStorage = new WxPayConfigStorage();
              wxPayConfigStorage.setMchId("合作者id（商户号）");
              wxPayConfigStorage.setAppid("应用id");
              wxPayConfigStorage.setKeyPublic("转账公钥，转账时必填");
              wxPayConfigStorage.setSecretKey("密钥");
              wxPayConfigStorage.setNotifyUrl("异步回调地址");
              wxPayConfigStorage.setReturnUrl("同步回调地址");
              wxPayConfigStorage.setSignType("签名方式");
              wxPayConfigStorage.setInputCharset("utf-8");
        
```

#### 网络请求配置

```java

        HttpConfigStorage httpConfigStorage = new HttpConfigStorage();
        /* 网路代理配置 根据需求进行设置**/
        //http代理地址
        httpConfigStorage.setHttpProxyHost("192.168.1.69");
        //代理端口
        httpConfigStorage.setHttpProxyPort(3308);
        //代理用户名
        httpConfigStorage.setHttpProxyUsername("user");
        //代理密码
        httpConfigStorage.setHttpProxyPassword("password");
        /* /网路代理配置 根据需求进行设置**/
    
        //退款使用
         /* 网络请求ssl证书 根据需求进行设置**/
        //设置ssl证书路径
        httpConfigStorage.setKeystorePath("证书绝对路径");
        //设置ssl证书对应的密码
        httpConfigStorage.setStorePassword("证书对应的密码");
        /* /网络请求ssl证书**/
      /* /网络请求连接池**/
        //最大连接数
        httpConfigStorage.setMaxTotal(20);
        //默认的每个路由的最大连接数
        httpConfigStorage.setDefaultMaxPerRoute(10);
```

#### 创建支付服务

```java 

        //支付服务
        PayService service =  new WxPayService(wxPayConfigStorage);
        
        //设置网络请求配置根据需求进行设置
        //service.setRequestTemplateConfigStorage(httpConfigStorage)

```

#### 创建支付订单信息

```java

        //支付订单基础信息
           PayOrder payOrder = new PayOrder("订单title", "摘要",  new BigDecimal(0.01) , UUID.randomUUID().toString().replace("-", ""));
  
``` 

#### 扫码付

```java

 
            /*-----------扫码付-------------------*/
           payOrder.setTransactionType(WxTransactionType.NATIVE);
           //获取扫码付的二维码
           BufferedImage image = service.genQrPay(payOrder);
           /*-----------/扫码付-------------------*/


``` 

#### APP支付

```java

        /*-----------APP-------------------*/
          payOrder.setTransactionType(WxTransactionType.APP);
              //获取APP支付所需的信息组，直接给app端就可使用
          Map appOrderInfo = service.orderInfo(payOrder);
        /*-----------/APP-------------------*/

``` 

#### 网页支付

```java

        /*-----------网页支付-------------------*/

        payOrder.setTransactionType(WxTransactionType.MWEB); //  网页支付
        //获取支付所需的信息
        Map directOrderInfo = service.orderInfo(payOrder);
        //获取表单提交对应的字符串，将其序列化到页面即可,
        String directHtml = service.buildRequest(directOrderInfo, MethodType.POST);
        /*-----------/即时到帐 WAP 网页支付-------------------*/

``` 

#### 条码付 刷卡付

```java

        /*-----------条码付 刷卡付-------------------*/
        payOrder.setTransactionType(WxTransactionType.MICROPAY);//条码付
        payOrder.setAuthCode("条码信息");
        // 支付结果
        Map params = service.microPay(payOrder);

        /*-----------/条码付 刷卡付-------------------*/

``` 

#### 回调处理

```java

        /*-----------回调处理-------------------*/
        //HttpServletRequest request;
         Map<String, Object> params = service.getParameter2Map(request.getParameterMap(), request.getInputStream());
        if (service.verify(params)){
            System.out.println("支付成功");
            return;
        }
        System.out.println("支付失败");


        /*-----------回调处理-------------------*/

```

#### 支付订单查询

```java
        
      Map result = service..query("微信单号", "我方系统单号");

```


#### 交易关闭接口
  ```java

          Map result = service..close("微信单号", "我方系统单号");

```


#### 申请退款接口
  ```java
        //过时方法
         //Map result = service.refund("微信单号", "我方系统单号", "退款金额", "订单总金额");
         //微信单号与我方系统单号二选一
         RefundOrder order = new RefundOrder("微信单号", "我方系统单号", "退款金额", "订单总金额");
         //可用于多次退款
         order.setRefundNo("退款单号")
         Map result = service.refund(order);

```


#### 查询退款
  ```java

          Map result = service.refundquery("微信单号", "我方系统单号");

```

#### 下载对账单
  ```java

          Map result = service.downloadbill("账单时间：日账单格式为yyyy-MM-dd，月账单格式为yyyy-MM", "账单类型");

```

#### 转账
>1. 转账到银行卡
  ```java
    
        order.setOutNo("partner_trade_no 商户转账订单号");
        //采用标准RSA算法，公钥由微信侧提供,将公钥信息配置在PayConfigStorage#setKeyPublic(String)
        order.setPayeeAccount("enc_bank_no 收款方银行卡号");
        order.setPayeeName("收款方用户名");
        order.setBank(WxBank.ABC);
        order.setRemark("转账备注, 非必填");
        order.setAmount(new BigDecimal(10));
        
        //转账到银行卡，这里默认值是转账到银行卡
         order.setTransferType(WxTransferType.PAY_BANK);
        Map result = service.transfer(order);

```
>2. 转账到余额
  ```java
    
        order.setOutNo("partner_trade_no 商户转账订单号");
        order.setPayeeAccount("用户openid");
        order.setPayeeName("收款用户姓名， 非必填，如果填写将强制验证收款人姓名");
        order.setRemark("转账备注, 非必填");
        order.setAmount(new BigDecimal(10));
        
        //转账到余额，这里默认值是转账到银行卡
        order.setTransferType(WxTransferType.TRANSFERS);
        Map result = service.transfer(order);

```

#### 转账查询
  ```java
       /**
       * wxTransferType 微信转账类型
       *                       {@link com.egzosn.pay.wx.bean.WxTransferType#QUERY_BANK}
       *                       {@link com.egzosn.pay.wx.bean.WxTransferType#GETTRANSFERINFO}
       */
       Map result = service.transferQuery("商户转账订单号", "wxTransferType  微信转账类型");
```
