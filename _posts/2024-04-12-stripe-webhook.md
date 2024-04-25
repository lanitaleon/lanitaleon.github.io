# Overview

Stripe Doc:

https://docs.stripe.com/webhooks

支付过程是异步的，需要 Stripe 通知我们的服务端。

Stripe 提供了两种通知方式：Webhook Config 和 Stripe CLI。

# Webhook Config

即在项目中开发一个接收事件通知的接口，并将该路径配置到 Dashboard > Developer > Webhook。

该方式仅支持回调 https 地址，本地开发可通过 Stripe-CLI 方式接收通知。

接收事件的接口示例：

```java
package com.stripe.sample;

import static spark.Spark.post;
import static spark.Spark.port;
import com.google.gson.JsonSyntaxException;
import com.stripe.Stripe;
import com.stripe.exception.SignatureVerificationException;
import com.stripe.model.*;
import com.stripe.net.Webhook;

public class Server {
    public static void main(String[] args) {
        // The library needs to be configured with your account's secret key.
        // Ensure the key is kept out of any version control system you might be using.
        Stripe.apiKey = "sk_test_...";
        
        // This is your Stripe CLI webhook secret for testing your endpoint locally.
        String endpointSecret = "whsec_59exxxxxxxxxx4f9bc232";

        port(4242);

        post("/webhook", (request, response) -> {
            String payload = request.body();
            String sigHeader = request.headers("Stripe-Signature");
            Event event = null;

            try {
                event = Webhook.constructEvent(
                    payload, sigHeader, endpointSecret
                );
            } catch (JsonSyntaxException e) {
                // Invalid payload
                response.status(400);
                return "";
            } catch (SignatureVerificationException e) {
                // Invalid signature
                response.status(400);
                return "";
            }

            // Deserialize the nested object inside the event
            EventDataObjectDeserializer dataObjectDeserializer = event.getDataObjectDeserializer();
            StripeObject stripeObject = null;
            if (dataObjectDeserializer.getObject().isPresent()) {
                stripeObject = dataObjectDeserializer.getObject().get();
            } else {
                // Deserialization failed, probably due to an API version mismatch.
                // Refer to the Javadoc documentation on `EventDataObjectDeserializer` for
                // instructions on how to handle this case, or return an error here.
            }
            // Handle the event
            System.out.println("Unhandled event type: " + event.getType());

            response.status(200);
            return "";
        });
    }
}
```

Dashboard 配置示例：

https://dashboard.stripe.com/webhooks

![webhook events](/img/stripe-webhook-events.png)

侦听事件不建议太多，否则要考虑自身服务端的吞吐量。

# Stripe CLI

Stripe Doc:

https://docs.stripe.com/stripe-cli/overview

本地开发或者是没有 https 域名的环境，可以启动该客户端替代 Stripe Webhook 功能。

与 Webhook 不同的是，该转发会转发所有事件，不可选择。

# Event Object

Stripe API Doc:

https://docs.stripe.com/api/events/object

一个例子：

```json
{
 "id": "evt_1NG8Du2eZvKYlo2CUI79vXWy",
 "object": "event",
 "api_version": "2019-02-19",
 "created": 1686089970,
 "data": {
   "object": {
     // different related object
   }
 },
 "livemode": false,
 "pending_webhooks": 0,
 "request": {
   "id": null,
   "idempotency_key": null
 },
 "type": "setup_intent.created"
}
```

在这个例子中只需要关注三个内容：

- type: 事件唯一标识

- data: 事件关联的数据对象内容

- live mode: 区分 Test mode 和 Live mode （生产模式）

# How to choose

开发人员需要仔细了解每个事件的描述，便于更好的决策。

文档中针对 Event 类型的描述，需要重点关注：触发时机和包含的对象类型。

除此之外，还需要注意：

1. 同一个类型的对象，在不同的 event 中，对象信息可能存在不同程度的缺失。

比如 invoice 状态为 draft 时，回调 event data 中必然不会包含 invoice number，invoice url 等信息。

2. 不同的对象，在不同的 event 中，可能代表了相同的业务含义。

比如在业务中，如果每一条 Checkout Session 都开启了 Invoice Creation，那么 checkout.session.complete 和 invoice.paid 对系统来说都意味着账单被支付了。

但是，如果业务系统还使用了 Setup 流程，那么 checkout.session.complete 可能还是需要订阅的，当然也可以选择直接订阅 setup_intent.succeeded。

即需要关注 webhook 订阅事件之间是否存在过度重叠，非必要不增订阅。

TBD - Show some json examples

# Automation

有一些事件不会自动触发，需要在 Dashboard 配置 Automation。

例如 invoice.overdue 的事件通知，默认不会触发，需要配置 Automation 触发。

另外，Automation 模块需要升级账号，过路费比普通等级稍贵。

![automation](/img/stripe-automation-upgrade.png)






