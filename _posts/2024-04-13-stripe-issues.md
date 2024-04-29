# checkout

1. checkout session 填写有问题的 credit card number,stripe会校验吗

会,会显示异常信息,且不能提交.

2. checkout 可以仅收集到支付方式、地址名称邮件,不经过支付吗?

可以,通过 checkout.setup mode 可以预收集支付信息.

或者通过自定义程度更高的 stripe element 进行集成;

setup mode 收集的支付方式不会自动变成 default_payment_method,需要再设置一次;

哪怕只有一个支付方式,也不会自动设成 default_payment_method,不能自动扣款;

https://stripe.com/docs/payments/save-card-without-authentication

https://stripe.com/docs/payments/save-and-reuse?platform=web

https://github.com/stripe/elements-examples/blob/master/js/example5.js

https://stripe.com/docs/elements/address-element

https://stripe.com/docs/payments/payment-element

https://stripe.com/docs/js/elements_object/create_element?type=card

2.1 default_payment_method

dashboard 可以手动设置,API里 invoice_settings.default_payment_method;

https://stripe.com/docs/payments/cards/overview

创建 invoice 的时候可以设一个 default payment method;

https://stripe.com/docs/api/invoices/create

2.2 更新 payment_method 只支持如下属性,要改更多只能删了再加;

name/address/expiration date/meta data;

尤其 card 卡号,cvc 等信息是绝对拿不到的,无法 clone;

3. checkout 可以生成 invoice 吗

可以, 设置参数 invoice_creation = true;

开启后 customer 也会以非 guest 的形式保存到 stripe,dashboard 可管理;

4. no-cost order 会收税吗?stripe会校验credit card吗?

不会,no-cost 不需要填 credit card,页面上只需要填邮箱（没开客户信息收集）;

5. checkout 支持 bank transfer 吗

checkout payment mode 支持,subscription mode 不支持;

https://stripe.com/docs/payments/bank-transfers/accept-a-payment

https://stripe.com/docs/payments/customer-balance/reconciliation

subscription mode 如果是自定义的话,通过 payment intent API 实现收款是支持的;

看下边这个文档中的 bank transfers 的支持情况;

https://stripe.com/docs/payments/payment-methods/integration-options

6. checkout hosted URL 只创建一次,如果链接失效才能重新创建;

在 webhook 中根据session id 查找生成过的 url,修改 status;

session status: open/complete/expired;

https://stripe.com/docs/api/checkout/sessions/object#checkout_session_object-status

7. checkout 携带 customer id,用户不使用默认支付方式,点切换填了新的,新的这个方式能自动绑定到 customer 上吗?

不可以.

直接用 payment_method_id attach customer_id 是不行的;

clone create 也不行,因为CVC安全原因拿不到;

This PaymentMethod was previously used without being attached to a Customer or was detached from a Customer, and may not be used again.

https://stripe.com/docs/payments/checkout/subscriptions/update-payment-details

这个文档也没用,但是提到了 subscription 可以有自己的 default payment method, 区别于 invoice.default_payment_method;

8. checkout url expire 有通知吗?

https://stripe.com/docs/payments/checkout/managing-limited-inventory

有通知.

9. 用户可以在账单或者邮件中取消支付吗?如何触发 payment_intent.canceled event?

https://stripe.com/docs/api/payment_intents/cancel

通过 api 可以取消支付,没找到用户自己取消的方式;

checkout 设置 cancel_url 页面上会显示返回按钮,但是点返回按钮是不会触发这个事件的;

10. payment intent 和 setup intent status 流转

https://docs.stripe.com/payments/paymentintents/lifecycle#intent-statuses

11. setup mode 不能收集税号

12. setup 的 customer 本身没有绑定 email 的时候,setup页面会展示 email输入框

# subscription

13. subscription 可以同时包含周期和一次性产品吗?

可以,subscription 只需要 items 包含一个周期产品,其他的items 随便;

其他的items 被包含在这条 invoice 中,但是不包含在后续的 subscription 中;

14. subscription 和 one-time product 在支付样式上有区别

subscription 支付页面会显示付款周期等信息;

15. 变更 subscription 的差价会在 stripe customer 生成余额;

16. subscription 切换不同产品时,周期中已经用掉的金额的计算方式可以自定义吗?

https://stripe.com/docs/billing/subscriptions/prorations

可以,默认是均分计算,可以关掉收全款;

17. coupon & promotion code

https://docs.stripe.com/billing/subscriptions/coupons

# invoice

18. auto-charge 自动扣款

draft >> open 有约 1h 的延时;

即使调用 finalize invoice api 也不会马上执行 auto-charge;

https://stripe.com/docs/invoicing/integration/automatic-advancement-collection

https://stripe.com/docs/payments/payment-methods/overview

subscription 可以选择：

从设置的 default payment method 尝试扣款,或者生成 invoice;

https://stripe.com/docs/billing/invoices/subscription

https://stripe.com/docs/invoicing/automatic-charging

自动扣款结束会 stripe 会发 receipts;

https://docs.stripe.com/invoicing/send-email

https://dashboard.stripe.com/settings/emails

第一个文档中 Email PDF attachment 提到：

If you turn on emails for successful payments—and an invoice is set to charge automatically—the receipt email includes a PDF attachment of both the original invoice and the invoice receipt. 

收据中会包含原始 Invoice;

该 PDF 可以在如下配置中关闭

https://dashboard.stripe.com/settings/billing/invoice?tab=general

19. invoice.finalization_failed vs invoice.finalization

https://stripe.com/docs/invoicing/integration/workflow-transitions

这俩都发生在 open 之前,指的是invoice是不是创建失败了和是不是准备好了被paid;

还没到支付那一步.

20. auto charge 重试全部失败,发送的邮件中 invoice url 有失效时间吗

没有,可以重新支付;

# customer

21. clone customer across account 不是关联的子账户能实现吗?

相关功能是 connect clone,还没试过;

https://stripe.com/docs/connect/cloning-customers-across-accounts

22. address validation

US state codes: https://en.wikipedia.org/wiki/List_of_U.S._state_and_territory_abbreviations

stripe API对地址信息没什么校验,,,

https://docs.stripe.com/tax/customer-locations#supported-formats

# tax

23. reverse-charge

https://stripe.com/docs/billing/taxes/tax-rates

https://stripe.com/docs/tax/zero-tax

https://www.sumup.com/en-gb/invoices/invoicing-essentials/reverse-charge-invoice/

24. reverse tax 的样式到底怎么弄成标注那种`[1]`小字的?

customer 直接配成 reverse charge 会变成一行说明,不是标注;

invoice 创建时参数 AutomaticTax 设成 true,

且 customer 不是 reverse charge,

且 客户地址和配置的 tax Id 不在一个国家,且是 eur,

https://docs.stripe.com/invoicing/customer/tax-ids

且不需要手动添加税率;

```java
InvoiceCreateParams invoiceParams = InvoiceCreateParams.builder()
        .setCustomer("cus_Paxxx1nsR")
        .setCollectionMethod(InvoiceCreateParams.CollectionMethod.CHARGE_AUTOMATICALLY)
        .setAutomaticTax(InvoiceCreateParams.AutomaticTax.builder()
                .setEnabled(true)
                .build())
        .setAutoAdvance(true).build();
Invoice invoice = Invoice.create(invoiceParams);

InvoiceItemCreateParams invoiceItemParams = InvoiceItemCreateParams.builder()
        .setCustomer("cus_Paxxx1nsR")
        .setPrice("price_1Oxxxx1eKvHg")
        .setInvoice(invoice.getId()).build();
InvoiceItem invoiceItem = InvoiceItem.create(invoiceItemParams);
``` 

如果不想显示标注,需要设置产品的tax code

https://docs.stripe.com/tax/tax-codes

# payment method

25. bank transfer 

线下通过银行转账付款,中间会经过不确定的流程,从而损耗手续费;

手续费在支付时可选谁承担,如果不是支付者承担就会导致到账金额不足;

26. bank debits

https://stripe.com/docs/payments/bank-debits

地域限制

27. bank transfer 场景中,每个 customer 有自己独立的虚拟银行卡号;

读取虚拟卡号的 API 是 funding instructions

https://docs.stripe.com/api/issuing/funding_instructions/object

但是 stripe 未开放 java lib

# api

28. currency

支持的货币

https://stripe.com/docs/currencies

unit_amount

最终显示的价格和发给 stripe 的价格 unit_amount 之间需要换算

https://stripe.com/docs/currencies?presentment-currency=ES#zero-decimal

默认2位,比如 1 USD/EUR/GBP/HKD = 100 amount

零位十进制 1 JPY = 1 amount

USD, EUR, GBP,JPY, HKD

USD - 0.5, EUR - 0.5, GBP - 0.3, HKD - 4, JPY - 50

对应2位country code: us/es/gb/hk/jp

29. the maximum number of payment methods 

that you can attach to a customer is up to 200.

https://docs.stripe.com/error-codes#customer-max-payment-methods 

# overdue

30. invoice.send_invoice 设置的 due days 过后,状态会变成什么

还是 open, dashboard 显示为过期,链接依然可以正常支付,支付完变成已支付;

31. auto charge 没有 overdue 的概念

所以通过配置 retry周期 超过10天 达成这个作用

注意 retry subscription/invoice 配置分开的;

配置1天后重试,15号 finalize 的账单,15号当天会重试,16号会重试;

32. overdue automation webhook

3-29 创建的 invoice, 设置 days_until_due=1, 日志显示 3-30 推送了通知;

# webhook

33. invoice.auto_charging 全部重试失败会收到什么 webhook events?

payment_failed

34. invoice.send_invoice 逾期未支付会收到什么 webhook events?

配置项中可设置 invoice 状态变更,默认值是保持 past_due ,这种情况下不会收到任何通知;

如果配置成 uncollectible 会收到 uncollectible event;

如果期望收到 invoice.overdue event 那么需要配一个 overdue automations;

https://dashboard.stripe.com/test/revenue_recovery/automations

https://docs.stripe.com/billing/revenue-recovery/automations

# email

35. stripe 的邮件内容可以定制吗?

https://dashboard.stripe.com/settings/billing/automatic

各种邮件提醒都可以有一定程度的定制,主要是选择一些处理类型;

https://support.stripe.com/questions/customizable-failed-payment-emails

https://stripe.com/docs/billing/revenue-recovery/customer-emails#customize-email

beta 版似乎可以定制标题什么的;

36. invoice auto charge 失败重试的邮件

https://docs.stripe.com/billing/collection-method

在 failed invoicing payments 这一节提到;

自动推进情况下,会标记成 uncollectible/past due, 这个在配置中可选;

https://docs.stripe.com/invoicing/automatic-collection#failed-payment-notifications

在 failed payment Notification 这一节提到;

一次性 invoice ,stripe 会发送 hosted invoice page link 给 customer;

也就是不会发修改 payment method 的链接,下次自动扣款还是失败的;

配置邮件页面：

https://dashboard.stripe.com/settings/billing/automatic

https://dashboard.stripe.com/revenue_recovery/retries

# customize

37. stripe element 页面深度自定义

https://stripe.com/docs/payments/quickstart

https://stripe.com/docs/security/guide#validating-pci-compliance

通过 payment intent api 走流程;

安全合规标准和风险自负;

38. invoice 多语言

customer preferred_locale

或者 invoice

https://docs.stripe.com/invoicing/customize

https://docs.stripe.com/api/customers/update

https://support.stripe.com/questions/language-options-for-customer-emails
