# 一、集成 Checkout

Stripe 产品概念分类：

![Stripe doc structure](/img/stripe-doc-structure.png)

Payment 侧重于即时支付，先创建支付会话，支付完成后生成账单和收据。

Billing 侧重于长周期支付，先创建账单，然后等待用户支付。

## 1.1 Overview

Stripe Checkout Doc :

https://docs.stripe.com/payments/checkout

https://docs.stripe.com/payments/checkout/how-checkout-works

Checkout API Doc: 

https://docs.stripe.com/api/checkout/sessions

https://docs.stripe.com/api/events/types

Checkout 的生命周期即 Checkout Session 的生命周期。

Session 的状态值范围：open/complete/expired

Session 的默认过期时间是 24H，该值可在创建 Session 时设置。

| Action | Session Status | Description |
| Create Session | open | 可支付状态 |
| Payment Successful | complete | 支付成功 |
| None / Expire Session | expired | 未支付或直接取消 |


Create Payment Session Example:

```java
SessionCreateParams params =  SessionCreateParams.builder()
   .setMode(SessionCreateParams.Mode.PAYMENT)
   .setUiMode(SessionCreateParams.UiMode.HOSTED)
   .addPaymentMethodType(SessionCreateParams.PaymentMethodType.CARD)
   .setCurrency("usd")
   .setCustomer("{{CUSTOMER_ID}}")
   .setSuccessUrl("https://example.com/success?session_id={CHECKOUT_SESSION_ID}")
   .setCancelUrl("https://example.com/cancel")
   .setInvoiceCreation(SessionCreateParams.InvoiceCreation.builder().setEnabled(true).build())
   .addAllLineItem(lineItems)
   .build();

Session session = Session.create(params);
```

Create Setup Session Example:

```java
SessionCreateParams params = SessionCreateParams.builder()
    .setMode(SessionCreateParams.Mode.SETUP)
    .setUiMode(SessionCreateParams.UiMode.HOSTED)    
    .addPaymentMethodType(SessionCreateParams.PaymentMethodType.CARD)
    .setSuccessUrl("https://example.com/success")
    .setCustomer("{{CUSTOMER_ID}}")
    .build();

Session session = Session.create(params);
```

其中的关键参数可参考 1.2 到 1.6 节的详细描述。

## 1.2 Mode

Checkout 支持三种 Mode ：PAYMENT / SUBSCRIPTION / SETUP

https://docs.stripe.com/api/checkout/sessions/create#create_checkout_session-mode

PAYMENT / SUBSCRIPTION 

指创建待支付的 Session。

如果产品列表中包含至少一个周期性的产品则必须使用 Subscription Mode。

如果产品列表均为一次性产品需使用 Payment Mode。

例如：

![Subscription Mode](/img/stripe-subscription-mode.png)

SETUP 

指创建收集支付方式的 Session，例如：

https://docs.stripe.com/payments/save-and-reuse

![Setup Mode](/img/stripe-setup-mode.png)

## 1.3 UI Mode

Checkout 页面默认支持两种类型：HOSTED / EMBEDDED

https://docs.stripe.com/api/checkout/sessions/create#create_checkout_session-ui_mode

### HOSTED 

指 Session 页面完全由 Stripe 托管，后端创建 Session 后会获得一个 URL；

前端直接打开该 URL 即可获得支付详情；

### EMBEDDED 

指将 Session 页面嵌入自己的系统页面中，后端创建 Session 后会获得一个 Client Secret Key；

前端需要集成 Stripe JS SDK，再通过 Client Secret Key 和 Stripe Publishable API Key 将 Session 表单嵌入自己的系统页面中。

https://docs.stripe.com/checkout/embedded/quickstart

React Example:

```js
const CheckoutForm = () => {
  const [clientSecret, setClientSecret] = useState('');

  useEffect(() => {
    // Create a Checkout Session as soon as the page loads
    fetch("/create-checkout-session", {
      method: "POST",
    })
      .then((res) => res.json())
      .then((data) => setClientSecret(data.clientSecret));
  }, []);

  return (
    <div id="checkout">
      {clientSecret && (
        <EmbeddedCheckoutProvider
          stripe={stripePromise}
          options={{clientSecret}}
        >
          <EmbeddedCheckout />
        </EmbeddedCheckoutProvider>
      )}
    </div>
  )
}
```

### cancel_url / success_url / return_url 

HOSTED 可配置 cancel_url 和 success_url。

cancel_url 非必填项，非空则页面左上角会出现【返回】按钮；

success_url 为必填项，支付完成后重定向的地址。

EMBEDDED 可配置 return_url，该字段为必填项，即支付完成后重定向的地址。

## 1.4 Payment Method Type

https://docs.stripe.com/api/checkout/sessions/create#create_checkout_session-payment_method_types

指定 Checkout Session 页面上可选择的支付方式，必须是 Stripe 配置中已开启的支付方式。

Tips:

SETUP Mode 不支持 CUSTOMER_BALANCE，即 Bank Transfer。

此处的 Bank Transfer 是指 Stripe 生成虚拟银行账号发送给用户，由用户自行向该账号转账，而非用户将自己的银行账户绑定到 Stripe Customer。

后者指的是 ACH Direct Debit：

https://docs.stripe.com/invoicing/ach-debit

## 1.5 Invoice Creation

Checkout Session 自成体系，默认不会生成 Invoice Object，需开启创建 Invoice 选项。

https://docs.stripe.com/api/checkout/sessions/create#create_checkout_session-invoice_creation

Tips: 

只有支付成功后才会自动生成 Invoice Object，未支付时不会生成。

```java
// SessionCreateParams params =  SessionCreateParams.builder()
 .setInvoiceCreation(SessionCreateParams.InvoiceCreation.builder().setEnabled(true).build())
//   .build();
```

## 1.6 Line Item

即 Product & Price List

https://docs.stripe.com/api/checkout/sessions/create#create_checkout_session-line_items

```java
List<SessionCreateParams.LineItem> items = new ArrayList<>();

SessionCreateParams.LineItem item = SessionCreateParams.LineItem.builder()
        .setPriceData(SessionCreateParams.LineItem.PriceData.builder()
                .setCurrency(currency)
                .setUnitAmountDecimal(BigDecimal.valueOf(10000L))
                .setProductData(SessionCreateParams.LineItem.PriceData.ProductData.builder()
                        .setName("product name")
                        .setDescription("product description").build())
                .build())
        .setQuantity(1L)
        .build();
items.add(item);

// SessionCreateParams params =  SessionCreateParams.builder()
     .addAllLineItem(lineItems)
//   .build();
```

## 1.7 Expire A Session

取消指定 Session，设置后该 Session 状态变更为 expired，不可支付。

https://docs.stripe.com/api/checkout/sessions/expire


```java
Session resource =
  Session.retrieve(
    "cs_test_a1Ae6ClgOkjygKwrf9B3L6ITtUuZW4Xx9FivL6DZYoYFdfAefQxsYpJJd3"
  );
SessionExpireParams params = SessionExpireParams.builder().build();
Session session = resource.expire(params);
```

## 1.8 No-Cost Orders

https://docs.stripe.com/payments/checkout/no-cost-orders

设置金额 unit_amount 为 0 可创建 0 元账单。

Tips: quantity 不能设为 0

![No-Cost Order](/img/stripe-no-cost-order.png)

## 1.9 Events

Checkout 流程中可能会生成如下对象：

Session，Setup Intent / Payment Intent，Invoice

相关 Events 如下所示，Events 之间存在信息重复，需要自行取舍。

Checkout Events

https://docs.stripe.com/api/events/types#event_types-checkout.session.async_payment_failed

Setup Intent Events

https://docs.stripe.com/api/events/types#event_types-setup_intent.canceled

Invoice Events

https://docs.stripe.com/api/events/types#event_types-invoice.created

Payment Intent Events

https://docs.stripe.com/api/events/types#event_types-payment_intent.amount_capturable_updated

