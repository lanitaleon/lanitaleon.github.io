# 二、集成 Invoice

Stripe Billing 模块主要分成 Subscription 和 Invoice。

Subscription 即 Checkout Session Mode = Subscription，先创建 Session，再生成周期性的账单 Invoice。

Invoice 是账单本体，支持两种收款方式：自动扣款、发送账单邮件。

## 2.1 Lifecycle

Stripe Invoice Doc:

https://docs.stripe.com/invoicing

https://docs.stripe.com/invoicing/overview

Stripe API Doc:

https://docs.stripe.com/api/invoices

https://docs.stripe.com/api/events/types

Invoice 的生命周期：

https://docs.stripe.com/invoicing/integration/workflow-transitions

![Invoice Lifecycle](/img/stripe-invoice-lifecycle.png)

Invoice 的状态集合：

draft, deleted, open, paid, uncollectible, void

https://docs.stripe.com/api/invoices/object#invoice_object-status

| Action | Invoice Status | Description |
| -- | -- | -- |
| Create A Invoice | draft | 初始状态，不可支付 |
| Delete A Draft Invoice | deleted | 最终状态，不可支付，彻底删除。 |
| Finalize A Draft Invoice | open | 中间状态，可支付，可变更为 paid/void |
| Payment Successful | paid | 最终状态，已支付 |
| Void An Open/Uncollectible Invoice | void | 最终状态，不可支付，账单已关闭 |
| All retries for a payment fail | uncollectible | 中间状态，可支付，可变更为 paid/void |

### open vs draft

这两个状态的 Invoice 在数据上的区别：

在 finalize 之前，Invoice Object 的 number, hosted_invoice_url, invoice_pdf 均为空值。

故无法进行支付流程。

### uncollectible

如何获得一个状态为 uncollectible 的 Invoice

1) 直接调用 API 指定 Invoice 状态

```java
InvoiceMarkUncollectibleParams params = InvoiceMarkUncollectibleParams.builder().build();
Invoice invoice = resource.markUncollectible(params);
```

2) 在 Stripe Dashboard 可配置延期未支付账单的后续行为：

配置指路：

Billing > Manage failed payments for subscriptions > Invoice Status

https://dashboard.stripe.com/settings/billing/automatic

Leave the invoice past-due 指的是 Invoice 保持 open 状态。

此时，在 Stripe Dashboard Invoice 列表中会出现 past-due 的红色标签，但其实没有 past-due 这个状态。

如果该配置改为第二项，则账单逾期未支付会自动变更为中间状态 uncollectible。

接下来介绍常用的 API 和一些使用场景。

## 2.2 Create Invoice

Quickstart Guide:

https://docs.stripe.com/invoicing/quickstart-guide

Stripe API Doc:

https://docs.stripe.com/api/invoices/create

Invoice 创建以后状态为 draft，此时可任意变更 invoice 的内容，变更 products & prices 等。

API 示例：

1) Auto Charge Example

```java
InvoiceCreateParams invoiceParams = InvoiceCreateParams.builder()
        .setCustomer("cus_PxxxxxawSn")
        .setCollectionMethod(InvoiceCreateParams.CollectionMethod.CHARGE_AUTOMATICALLY)
        .setAutoAdvance(true).build();

Invoice invoice = Invoice.create(invoiceParams);
```

2) Send Email & Bank Transfer Example

```java
InvoiceCreateParams params = InvoiceCreateParams.builder()
    .setCustomer("cus_PxxxxxawSn")
    .setPaymentSettings(InvoiceCreateParams.PaymentSettings.builder()
            .addPaymentMethodType(
                    InvoiceCreateParams.PaymentSettings.PaymentMethodType.CUSTOMER_BALANCE
            )
            .setPaymentMethodOptions(InvoiceCreateParams.PaymentSettings.PaymentMethodOptions.builder()
                    .setCustomerBalance(InvoiceCreateParams.PaymentSettings.PaymentMethodOptions.CustomerBalance.builder()
                            .setFundingType("bank_transfer")
                            .setBankTransfer(InvoiceCreateParams.PaymentSettings.PaymentMethodOptions.CustomerBalance.BankTransfer.builder()
                                    .setType("gb_bank_transfer").build()
                            ).build()
                    ).build()
            ).build()
    )
    .setCollectionMethod(InvoiceCreateParams.CollectionMethod.SEND_INVOICE)
    .setDaysUntilDue(10L)
    .setAutoAdvance(true)
    .build()
Invoice invoice = Invoice.create(invoiceParams); 
```

Invoice 账单详情页由 Stripe 托管，样式参考：

1) Payment Method Type - Card

![Hosted Invoice Page - Card](/img/stripe-invoice-hosted-page-card.png)

2) Payment Method Type - Bank Transfer

![Hosted Invoice Page - Bank Transfer](/img/stripe-invoice-hosted-page-bank-transfer.png)

### Invoice Item

Stripe API Doc:

https://docs.stripe.com/api/invoices/line_item

与 Create Checkout 不同的是，Create Invoice 有两步：

a. 创建 Invoice Object 

b. 创建 Invoice Items 即产品价格列表，并绑定到指定的 Invoice。

```java
PriceCreateParams priceParams = PriceCreateParams.builder()
        .setCurrency("usd")
        .setUnitAmount(1000L)
        .setProductData(PriceCreateParams.ProductData.builder()
                .setName("product name").build()).build();
Price price = Price.create(priceParams);

InvoiceItemCreateParams invoiceItemParams = InvoiceItemCreateParams.builder()
        .setCustomer("cus_PxxxxxawSn")
        .setPrice(price.getId())
        .setDescription("product description")
        .setInvoice(invoice.getId()).build();
InvoiceItem invoiceItem = InvoiceItem.create(invoiceItemParams);
```

### Auto Advance

Stripe Doc:

https://docs.stripe.com/invoicing/integration/automatic-advancement-collection

Stripe API Doc:

https://docs.stripe.com/api/invoices/create#create_invoice-auto_advance

该参数指自动推进 Invoice 的状态变更。

如果 _auto_advance=false_，则需要指定行为才能触发状态变更，比如 Draft Invoice 在调用 Finalize Invoice 后才会变成 Open Invoice。

如果 _auto_advance=true_，则 Stripe 将在 1 小时后自动将 Invoice 状态变更为 open。

Tips:

在调用 Finalize An Invoice API （小节 2.4）时，也需要设置该参数。

### Collection Method

Stripe Doc:

https://docs.stripe.com/invoicing/automated-collections

Stripe API Doc:

https://docs.stripe.com/api/invoices/create#create_invoice-collection_method

该参数指收款方式，有两种收款方式可选：自动扣款、账单邮件。

1) charge_automatically

向 Customer 设置的默认支付方式扣款。

可在 Stripe Dashboard 配置重试周期、重试次数等。

2) send_invoice

向 Customer Email Address 发送账单邮件。

可设置账单逾期时间。

在 Stripe Dashboard 可配置支付提醒周期。

3) Billing Settings

配置指路：

Billing > Subscription and emails > Manage advanced invoicing features > Customer emails

https://dashboard.stripe.com/settings/billing/automatic

![Billing Settings](/img/stripe-invoice-retry-config.png)

## 2.3 Delete Draft Invoice

Stripe API Doc:

https://docs.stripe.com/api/invoices/delete

仅可删除状态为 draft 的 Invoice，该删除为彻底删除，不可恢复。

删除后，在 Stripe Dashboard 无法查看该 Invoice，调用 Retrieve Invoice API 也无法查到该 Invoice。

```java
Invoice resource = Invoice.retrieve("in_1MxxxxxxxPMv");
Invoice invoice = resource.delete(); 
```

## 2.4 Finalize Invoice

Stripe API Doc:

https://docs.stripe.com/api/invoices/finalize

Invoice 数据装填完成后，可调用该 API 将 Invoice 变更为 open 状态，即变为可支付状态。

```java
Invoice resource = Invoice.retrieve("in_1MtGxxxxx6PgS6g8S");
InvoiceFinalizeInvoiceParams params = InvoiceFinalizeInvoiceParams.builder().setAutoAdvance(true).build();
Invoice invoice = resource.finalizeInvoice(params);
```

_auto_advance=true_，指 Invoice 状态由 Stripe 自动推进 （ 小节 2.2.2 Auto Advance ）。

## 2.5 Void Invoice

Stripe API Doc:

https://docs.stripe.com/api/invoices/void

非 draft Invoice 如果需要取消，不再允许支付，可调用此接口。

```java
Invoice resource = Invoice.retrieve("in_1MtGmxxxxxgS6g8S");
InvoiceVoidInvoiceParams params = InvoiceVoidInvoiceParams.builder().build();
Invoice invoice = resource.voidInvoice(params);
```

与 2.3 Delete Draft Invoice 的区别是：

Void 不可用于 Draft Invoice。

Void Invoice 在 Stripe Dashboard 可见，调用 Retrieve API 可查询，Hosted invoice page 可访问。

## 2.6 Retrieve A Invoice

Stripe API Doc:

https://docs.stripe.com/api/invoices/retrieve

```java
Invoice invoice = Invoice.retrieve("in_1MtHxxxxxPMv");
```

**Tips**

Status 为 deleted 的 Invoice 已被彻底删除，不可查。

调用 Retrieve API 将抛出如下异常：

```java
Exception in thread "main" com.stripe.exception.InvalidRequestException: No such invoice: 'in_1OoJHxxxxxx66lX'; code: resource_missing; request-id: req_EVxxxxxJ5k
  at com.stripe.net.LiveStripeResponseGetter.handleApiError(LiveStripeResponseGetter.java:182)
  at com.stripe.net.LiveStripeResponseGetter.handleError(LiveStripeResponseGetter.java:143)
  at com.stripe.net.LiveStripeResponseGetter.request(LiveStripeResponseGetter.java:67)
  at com.stripe.model.Invoice.retrieve(Invoice.java:1307)
  at com.stripe.model.Invoice.retrieve(Invoice.java:1294)
  at com.stripe.sample.InvoiceServer.main(InvoiceServer.java:119)
```

