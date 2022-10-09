####

#### Entity 实体 
例如，
    你的计算机是一个 entity，
    你写的代码也是一个 entity，
    你自己也是一个 entity，
    你吃的杂粮饼也是一个 entity，
    你六岁时见过的幽灵也是一个 entity —— 即使你妈告诉你幽灵并不存在，这只是你的臆想。
#### Identity  身份

每个 entity（实体）都有一个 identity（身份）。要精确定义这个概念比较困难，这么来说吧：identity 是使你之所以为你（what makes you you）的东西，懂吗？

具体到计算机领域，identity 通常用一系列属性来表示，描述某个具体的 entity，这里的属性包括 group、age、location、favorite color、shoe size 等等。

#### Identifier（身份标识符）

Identifier 跟 identity 还不是一个东西：每个 identifier 都是一个唯一标识符，也唯一地关联到某个有 identity 的 entity。

例如，我是 Mike，但 Mike 并不是我的 identity，而只是个 name —— 虽然二者在我们小范围的讨论中是同义的。
#### Claim（声明） & Authentication（认证）

    一个 entity 能 claim（声明）说，它拥有某个或某些 name。

    其他 entity 能够对这个 claim 进行认证（authenticate），以确认这份声明的真假。

    一般来说，认证的目的是确认某些 claim 的合法性。

    Claim 不是只能关联到 name，还可以关联到别的东西。例如，我能 claim 任何东西：my age, your age, access rights, the meaning of life 等等。

##### Subscriber & CA & relying party (RP)

    能作为一个证书的 subject 的 entity，称为 subscriber（证书 owner）或 end entity。

    对应地，subscriber 的证书有时也称为 end entity certificates 或 leaf certificates，原因在后面讨论 certificate chains 时会介绍。

    CA（certificate authority，证书权威）是给 subscriber 颁发证书的 entity，是一种 certificate issuer（证书颁发者）。

    CA 的证书，通常称为 root certificate 或 intermediate certificate，具体取决于 CA 类型。

    Relying party 是 使用证书的用户（certificate user），它验证由 CA 颁发（给 subscriber）的证书是否合法。

    一个 entity 可以同时是一个 subscriber 和一个 relying party。也就是说，单个 entity 既有自己的证书，又使用其他证书来认证 remote peers，例如双向 TLS（mutual TLS，mTLS）场景。

