# 可靠数据传输　Reliable Data Delivery　
可靠数据传输是系统的属性之一。就像性能一样，在设计之初就必须考虑进来。不可能在系统完成后添加进来。可靠性是一个系统的资产，当我们谈到kafka保证可靠性，你必须思考整个系统和实际场景。整合kafka的系统和kafka自身一样重要。因为可靠性不仅仅是一个人的职责。每个人包括kafka管理员，linux管理员，网络和存储管理员和应用开发者必须同心协力一起构建一个可靠系统。
Kafka对于可靠数据传输是非常灵活的。Kafka有很多使用场景，从跟踪网站点击到信用卡支付。一些场景需要极致的可靠同时另一些是速度和简单性大于可靠性。Kafka提供了足够的配置项和灵活的客户端(client API)足以允许在任何场景中权衡。
因为它的灵活，也容易弄伤自己，当你认为系统需要可靠而实际并不需要时。我们先讨论不同的可靠性和他们在kafka中意味着什么。然后介绍副本机制和对可靠性有什么帮助。再讨论客户端，生产端，消费端在不同场景中如何使用。最后我们需要验证系统的可靠性而不仅仅相信它是可靠的。

## 可靠性保证