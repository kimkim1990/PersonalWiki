## [轉載] 前端服务化——页面搭建工具的死与生 [Back](./../post.md)

> - Author: [sskyy](https://github.com/sskyy)
> - Origin: http://www.cnblogs.com/sskyy/p/6496287.html
> - Time: Mar, 3rd, 2017

### 引言

我有个非常犀利的朋友，在得知我要去做可视化的页面搭建工具时问了我一个问题： “你自己会用这样的工具吗？” 同时带着意味深长的笑。

然而这个问题并没有如他所愿改变我的想法。早在 jquery ui、bootstrap 盛行的时代，就有过无数这样的工具，我没有用过，也不会去用。原因有一万个:

- 业务需求太灵活，工具都是基于已有的组件库，个性化的东西搭不出来。
- 前端技术发展太快，工具整合没有那么迅速。
- 有学习成本，不如手写快，灵活性没有手写高。

在包括我的很多前端看来，这条路上尸骨累累，甚至有很多连痕迹都没有留下。但是失败者最多的路，并不一定是死路。如果都没有抛开过头脑里的成见，没有进行过独立思考就放弃了，未免太盲目。这篇文章就当做我在求生之路上的记录。也请读者暂且忘掉所有的经验，轻装上阵，这趟旅途不会让你失望。对具体设计不感兴趣的读者可以直接阅读《生门》一章，读完那一章后或许你会迫不及待再从头读起。

### 起点和方向

在接下来的两章中，我们将从项目背景一直讨论到关键技术的实践。这其中既会包括各种技术也会包括产品和交互的思考。 项目的背景是，公司业务迅速扩张，有大量对内的系统页面需要搭建。而前端人力是瓶颈，所以我们希望能以服务化的方式输出前端能力，让公司内所有非前端出身但有编程能力的人都能使用这种服务快速地开发出较高质量的页面。

从产品角度来说，它的目标已经很明确了：

- 使用人群：非前端的开发者
- 要提供的服务：能以中上等开发速度开发出中上等可维护性页面的集成开发环境(以下简称开发环境)

有了这个目标，我们就可以开始设计产品形态了。

页面分为视图和逻辑两部分，在目前组件化的大背景下，视图基本上可以等同于组件树。首先，什么样的页面编辑方式学习成本最低同时最快速？当然是所见即所得，拖拽或者编辑树型结构的数据这两种方式都可以接受。实际测试中拖拽最容易上手，熟悉了快捷键的情况下则编辑组件树更快。

接着，怎样让用户编写页面逻辑既能学习成本低，又能保障质量？学习成本低意味着概念要少，或者都是用户已知的概念。保障质量这个概念比较大，我们可以从开发的两个阶段来考虑：

- 一是开发时最好有保障，例如前端开发时的 eslint 加上编辑器提示就能很好地提前避免一些低级错误。
- 二是在开发完之后，代码应该“有迹可循”，无论是排查问题，还是扩展需求，都要让用户在头脑里第一时间就知道应该怎样写逻辑，写在哪里。也就意味着概念要完善，职责分明。同时，工具层面也可以有些辅助功能，例如传统编辑器的变量搜索等。

为了给读者一个更直观的影响，我们暂且来看一张两张图。

页面编辑:

<p align="center">
    <img src="https://zos.alipayobjects.com/rmsportal/blelrkgUdVBVTucDuuoT.png" alt="页面编辑"></img>
</p>

逻辑编辑:

<p align="center">
    <img src="https://zos.alipayobjects.com/rmsportal/sgarfasRwViOjDvvePUv.png" alt="逻辑编辑"></img>
</p>

接下来分部分细化形态，梳理关系，来得到一个明确的架构图。

目前看来可先拆分成三个部分:

- 一个编辑页面和逻辑的工具，以下暂称 IDE。
- 搭建页面所需的基础组件。
- 运行时框架(以下简称框架)，由它将页面的组件树、和页面逻辑结合在一起渲染成最终的页面。

很容易发现这三者的关系并不是平行的。首先，IDE 在这三者中是直接给用户使用的产物，它代表着我们最终想要呈现给用户什么样的东西。对其他部分来说，它算是需求来源。

来看它和页面以及组件的的关系。我们最终希望用户在点击页面上的某个组件或者组件树上的节点时，就能查看、配置这个组件上的属性，逻辑绑定到它触发的事件上。

<p align="center">
    <img src="https://zos.alipayobjects.com/rmsportal/uMMIizInErTFLUCCSRgQ.png" alt="组件与属性"></img>
</p>

因此它对组件的需求是：组件必须暴露出自己的所有属性和事件，让外部可读。

再看 IDE 和框架的关系。用户在编写逻辑时，需要理解的概念都是属于框架的，IDE 只是编辑工具。当然 IDE 可以提供很多辅助功能，例如语法校验，例如可视化地展示逻辑与组件的绑定关系。框架为主，IDE 为辅。

最后，框架和组件的关系。这里很有意思，按技术发展的现状来说，一直都是先有组件库，才有上层应用框架。然而，组件规范其实应该是应用框架规范的一部分。举个实际例子，如果应用框架要建立全局数据源(方便做回滚等高级功能)，来保存所有状态。那么组件就不再需要内部状态，只要渲染就够了，实现上简单很多。这种上层建筑与基础设施的关系，很像高楼与砖瓦。摩天大楼需要钢筋混凝土，负责烧土砖的工人一开始是想不到的。所以实施中，框架和组件库之间通常还会有适配层。优秀的架构能力就体现在一开始就看到了足够多的上层需求，提前避免了发展中的人力损耗。

理清了所有关系后，来看看整体架构：

<p align="center">
    <img src="https://zos.alipayobjects.com/rmsportal/QRMIwUZyhvduxcXkxkOu.png" alt="整体架构"></img>
</p>

这其中将 IDE 底层和业务层进行了拆分，IDE 底层提供窗口、快捷键、Tab 等常用功能，IDE 上业务层才用来处理和可视化相关的内容。其中也包括为了提供更好体验，却又不适合放到组件、和应用框架中的胶水代码，例如组件属性的说明，示例等等。IDE 的架构设计将会在另一篇文章中介绍。

### 龙骨

整体的架构有了后，接下来就是关键技术——运行时框架的设计了。 在数据驱动的大背景下，应用框架处理的问题实际上只有一个：数据管理。其中“数据”既包括组件数据也包括业务数据，而“管理”既包括如何保存数据，也包括以何种方式让用户来读写数据。我们仍然从使用场景出发，来分析出数据管理的应用场景，最后再考虑设计实现。在前端领域内，用户对交互的需求是渐进增长的，业务的需求是渐进的，因此应用的复杂度整体看来也是渐进的。所以我们只需要明确出最简单和最复杂的情况，就可以勾勒出框架需要支持的范围了：

- 按业务经验，最简单的情况无非就是纯展示的“详情页” 或 “列表页”。最符合本能的逻辑写法应该是:
- 拼好组件树。如果是静态的数据，直接在每个组件上的属性里设置好即可，流程结束。
- 如果是动态数据，那么使用 ajax 获取到数据。将获取的数据格式化成组件所能接受的格式，然后使用 api set 进去。

在这个场景中用户需要了解两件事情:

- 组件的数据格式，实际上就是组件的属性，在 IDE 中已经是直接暴露出来的。
- 设置数据的 api 。 再接着看最复杂的场景，我所接触过的最复杂的前端应用都是业务关联极强的工具，例如云计算平台的控制台，客服系统的控制台，包括这个 IDE 也算。

这类产品的复杂体现在两个方面:

- 有大量的交互细节，例如组件状态要和权限结合(例如 按钮的 disable 状态)、组件要根据需求动态显示或隐藏，表单的校验，异步状态的提示或管理(例如发送请求后，按钮上出现loading)。
- 除了组件数据，还有大量的业务数据要管理，并且是其中有很多联动关系。例如在云计算控制台里面有 ECS、LBS 等概念，ECS 和 LBS 有关联关系，ECS如果改名了。不仅要更新ECS自己的详情显示，还要自动更新关联的LBS的显示等。

有了这两个端点，就找到了要提供的能力的上限和下限，接下来就是框架设计中最有意思也最困难的部分了——如何提供渐进式地开发体验。这几乎也是所有优秀框架的共有的一个品质。渐进式的体验意味着用户只要了解最基本的功能就能马上开始工作，当要处理更高级的需求时才需要再学习高级的功能。更进一步话，最好这些高级功能也是用一种可扩展的机制来实现的，如中间件，学习一次机制，即可解决无限的问题。

在最简场景里可以看到，用户所需的最基本的功能就是一个可读写的，包含所有组件数据的数据源即可(以下简称组件数据源)。为了便于让用户理解，这个数据源的数据格式最好与组件树存在类似的对应关系。举个注册页面的例子，我们的组件树可能长这样:

```html
<div>
    <title>注册</title>
    <input label="姓名">
    <input label="密码" type="password">
    <button text="提交"></button>
</div>
```

那么组件数据源可表述为:

```js
{
    0: { text: '注册', size: 'large' },
    1: { value: '', label: '姓名', type: 'text' },
    2: { value: '', label: '密码', type: 'password' },
    3: { text: '提交', type: 'normal' }
}
```

用户的读写操作可以设计成这样:

```js
// 借用 redux 中的 store 作为数据源的名字
store.get('1.value') // 读取第一个 Input 的值
store.set('3.type', 'loading') // 将 Button 设为 loading 状态
```

这个写法可以实现需求，但有两个问题:
- 用组件的位置作为索引不友好，不能适应变化。例如组件的位置调整了一下顺序，代码里就得相应改动。
- 在用户的业务逻辑中，并不是所有组件的数据用户都需要，例如Title。

为何不让用户自己给想要数据的组件取名？这可以一次性解决这两个问题。

```html
<div>
    <title>注册</title>
    <input bind="name" label="姓名">
    <input bind="password" label="密码" type="password">
    <button bind="submit" text="提交"></button>
</div>
```

得到的数据源:

```js
{
    name: { value: '', label: '姓名', type: 'text' },
    password: { value: '', label: '密码', type: 'password' },
    submit: { text: '提交', disabled: false }
}
```

再看看用户的提交逻辑如何写(这个逻辑绑定在 Button 的 onClick 事件上):

```js
// 通过注入的方式把数据源管理对象交给用户
function({store}) {
    store.set('submit', {disabled: true}) // 为了防止重复提交
    ajax({
        url : 'xxx',
        data: {
            name: store.get('name').value,
            password: store.get('password').value
        }
    }).finally(() => {
        store.set('submit', {disabled: false})
    })
}
```

稍微好了一点，但是任何开发者都仍然会觉得这段代码太脏，它既处理了业务逻辑又处理了渲染逻辑，项目膨胀之后这样的代码不利于维护。

我们需要一种机制来分离不同类型的处理逻辑，让代码更易维护。这个出发点也正是启发后面设计的关键！ 为什么这样说？让我们来看看之前谈到的复杂场景，其中提到了大量的交互状态是复杂场景的特点之一，常见的交互有：

- 异步状态控制，如上面 button 在发请求时要设为 disable 防止重复提交
- 权限控制
- 表单验证状态

如何分离这些交互细节？或者换个更具体的问题，你觉得用户怎样写这些逻辑会最爽？仍然以上面的场景为例子，用户当然希望他代码中的ajax一发送，按钮就自动变成 disable，一结束又自动变回来。这对我们来说不就是 ajax 状态和组件状态之间的自动映射吗？我们能不能提供一种机制让用户给 ajax 命名，同时可以写映射关系，如:

```js
ajax('login', {name: 'xxx', password: 'xxx'})
```

映射关系:

```js
function mapAjaxToButton({ajaxStates}) {
    // ajaxStates 由框架提供，保存着所有的ajax 状态
    return { disabled: ajaxStates.login === 'pending' }
}
```

这样，刚才处理 ajax 的脏代码就完全分离出来了。我们再看看这个方案中几个概念的关系。

<p align="center">
    <img src="https://zos.alipayobjects.com/rmsportal/RatpXXxLIEZaejNQynnK.png" alt="数据源架构1"></img>
</p>

打开这个思路后，你会发现几乎其他所有问题，都可以用这个方案来解了！为专有的问题领域建立专有的数据源，同时建立数据源到组件数据源的映射关系。即能扩展能力，又能分离代码。

我们再看权限控制的例子。如果用户不具有某权限时就把button disable 掉，映射关系我们可以写成:

```js
function mapAuthToButton({auth}) {
    return { disabled: !auth.has('xxx') }
}
```

非常直观。 再看表单验证状态。建立验证数据结果的数据源，让用户配置哪些组件需要进行校验，校验时机(例如正在输入或者离开焦点时)。例如：

```html
<input bind="name" onblur={state => {validation.validate(state)}} />
```

validator 映射写法的和前面的例子异区同工，用户希望的当然是我只需要告诉你什么情况下是通过，什么不通过即可，同时也可以加上一些必要的message:

```js
function validateRule(state) {
    return {
        valid: state.value !== 'xxx',
        message: state.value !== 'xxx' ? 'success' : 'value must be xxx'
    }
}
```

有了输入源，接下来仍然按之前思路将验证数据源映射到组件数据源上:

```js
function mapValidationToInput({validation}) {
    const hasFeedback = validation.get('name') !== undefined
    return {
        status: hasFeedback ? (validation.get('name').valid ? 'valid': 'invalid') : 'normal',
        help: hasFeedback ? validation.get('name').message : ''
    }
}
```

到这里，我们已经完全看到用专属的数据源处理专有问题，最后映射到组件数据源上去所产生的效果了。它能很好地将所有将交互细节和业务逻辑划分。

我们进一步注意到，无论异步控制、表单验证还是权限，只要组件遵循某种属性命名规则，那么所有的映射函数就都可以写成固定的！

因此，如果我们为组件制定一个属性接口规范，就可以利用提供更有好的方式自动生成映射代码了。例如，规定带验证功能的表单类的属性接口必须有:
- status: 'normal' | 'valid' | 'invalid'
- help : ''

那么上面例子里面的映射函数，就只需要用户填写 validateRule 就够了，映射函数将 valid/message 字段映射到组件的 status/help 属性上。

至此，最后剩下的处理复杂场景中的大量业务数据的这一问题也迎刃而解了，同样建立一个业务数据源，声明业务数据与组件数据的映射关系即可。

<p align="center">
    <img src="https://zos.alipayobjects.com/rmsportal/fYMQMhvVgwPrfyFVbugp.png" alt="数据源架构2"></img>
</p>

讲完了逻辑的设计，最后再提一下组件的规范，正如前面所说，所有的组件状态是由应用框架保存的。这和我们现实中常见的经验相悖。现实中的组件通常是数据、行为、渲染逻辑三部分写在一起，使用 class 或者工厂方法来创建。如果是全面由框架接管，则应该打散，全部写成声明式。虽然不符经验，但是声明式的组件定义解决了《理想的应用框架》中提到的组件库的两个终极问题，“覆写和扩展”。具体可参见以开源的组件规范 https://github.com/sskyy/react-lego ，这里不再展开。


### 生门！

在还没有开始项目之前玉伯就提醒过我，IDE做得再酷炫，组件做得再丰富都不是活路。可视化的集成框架真正的问题在于：虽然对没有前端能力的人来说，它更简单。但相比手写代码它缺少了灵活性，那么在用户前端能力增强后，你拿什么来补偿用户，让他仍然离不开你？这里我可以再清晰的回答一次。

任何一个有一定复杂度、会持续增长的应用最重视的，其实并不是开发速度，而是可维护性和可扩展性。 这也是框架设计者们摆在首位的事情。可扩展性的好坏取决于框架的扩展机制。在我们的上面的设计中需要扩展的有两部分，组件和功能。组件的扩展可以通过允许用户提交自定义组件来实现。功能的扩展主要由框架开发者完成，但是也可以考虑让用户能仿照异步管理数据源一样建立自己专用的数据源来解决业务专有问题。

可维护性，在数据驱动的前提下，实际上等于”框架能不能很好的回答两个问题“:

- 数据现在是什么样的
- 数据在哪里被修改了，或者更细致地分解为“运行时告诉我数据这次在哪里被修改了”，和“开发时告诉我数据有可能在哪里被修改”。

第一个问题容易解决，建立统一的全局数据源，正如我们所设计的。不仅方便调试，还可以做回滚，做应用快照等功能。

第二个问题，在已知的框架中有两种常见的答案：

一种是利用某种设计模式，让用户将数据的变化集中在一个抽象里。例如 redux 状态机中的 reducer。这种方式的好处在于直接看代码就可以了解数据所有可能发生的变化。但靠代码组织的问题在于它本身受文件系统影响，一但代码拆分不合理还是容易不好找。

另一种方式则更常见，就是运行时记录调用栈。在 《理想的应用框架》中也提到过。以”响应业务事件的声明式代码“作为基础单位，框架来控制调用流程，这样框架即可产出一个和业务事件一致的调用栈，同时因为这种一致性，无论代码拆分得多不合理，都可以展示合理的信息。但调用栈的方式也有个缺点，就是一定要运行，出问题时一定要运行到相应的那一步才能找到问题相应的信息。同时会受到循环、条件语句的影响，这在多步调试或者非幂等操作的场景下非常不好用。它只能回答“数据这次在哪里被修改了”，不能回答“数据都可能在哪里被修改”。

有没有一种方式，既是静态的，又能产出像调用栈一样的数据结构方便做辅助工具呢？当然有！语法分析就可以，它绝对准确，不受条件语句、异常等影响，甚至能做到提前预知人为错误。Rust 在提前预知人为错误这个方面上达到了一个新高度。它做到了”能编译通过就不会出错“ ，这让工程质量产生了质的提升。举个我们系统中可以理解的例子，在前面的设计中已经提到，组件是声明式的，所以数据格式是已知并且可读的，包括每个字段的类型。在实现中我们的后端使用了 graphQL 作为接口层，因此接口返回的数据结构和字段类型也是已知的，当用户在代码中调用后端接口并尝试把接口返回的数据塞到组件上来展示时，通过语法分析、变量追踪，我们就可以在“运行前”自动检测到用户是否传错了接口参数，是否把不符合组件数据格式的数据塞给了组件等等。这样强度的检测几乎可以帮我们避免日常开发中绝大多数人为失误。除了诊断，语法分析当然还能用来提供全局的依赖视图，例如哪些接口在哪些逻辑里被调用了。哪些数据被哪些逻辑修改了，会引起视图的哪些部分改变等等。可以完美地回答“数据在哪里被修改了” 。

接下来就是如何实现的问题了。稍微想想就会发现，基于手写代码的方式分析成本有点高，而且很有可能实现不了。这里面有两个点比较麻烦：
- 分析程序首先要理解基于文件系统的包管理机制，才能做全局的分析。
- 如果用户在代码中做了二次抽象，分析程序的复杂度会翻倍。试想分析 `store.set('xxx', 'yyy')` 和 分析 `store[method](name) `的复杂度。

但是，我们刚刚设计的系统不是放弃了灵活性吗？用户在使用 IDE 时不需要文件系统的概念，只需要如填空一般在函数中写逻辑，所有依赖的变量也不需要自己关系，都是框架通过函数参数注入的。在这个背景下，用户逻辑的目的提前知道了，所有的入参出参的用途也提前知道了，那么要实现上述的“数据在哪里被修改了”等功能，是不是只需要追踪用户代码里的变量就够了？！上面说的难点在我们这里不存在了。

到这里，死门竟然变成了生门！“开发环境通过对逻辑使用的限制，实现了对整个应用的控制达到了 100% 的状态“！具体可以从两个方面来进一步理解：
- ”对逻辑使用的限制“指的是具体做某件事的代码写在哪里，必须怎么写都是由开发环境完全指定的。这意味着开发环境完全控制了所有代码的语意背景。但同时也是因为这样，开发环境说做不了的事情，就一定做不了，限制了用户的自由发挥。
- ”控制达到 100%“ 指的是开发环境可以分析理解所有用户逻辑，你提的所有“什么数据/接口/组件，在哪里/什么时候，怎么了？”这样的问题它都可以回答。实际上 js 在这里只是一种DSL了。举几个更具体的说法来表示 100%：
    - 除了用户自己对业务理解的错误，开发环境几乎可以提前阻止所有人为失误，如前面所说的数据类型不匹配，ajax 参数错误等等。注意，这里说的是提前阻止，不需要到运行时调试才发现
        - 开发环境可将所有逻辑和其中的依赖可视化，例如可完整地列举出所有操作了某一数据的逻辑代码。
        - 开发环境有足够能力对用户代码进行自动升级转换等工作。例如将 js 里的所有数据操作自动变成 immutable，排除潜在的对象引用错误等。
        - 开发环境可以深度分析运行时框架，提前注入运行时数据，提升运行时性能。例如提前分析哪些数据修改会导致哪些组件属性，静态注入这种依赖关系，这样框架就不再需要运行时再去判断。这种数据到视图的依赖绑定也正是过去 MVVM 类框架花了很大力气去做的事情。

运行时分析示例:

<p align="center">
    <img src="https://zos.alipayobjects.com/rmsportal/OIfpEOEOBKztoNsioCpF.png" alt="运行时分析示例"></img>
</p>

静态依赖分析示例:

<p align="center">
    <img src="https://zos.alipayobjects.com/rmsportal/WUzCOpYpluTVHClaLSPj.png" alt="静态依赖分析示例"></img>
</p>

想到了这里，才算真正找到了活路。文章的前半部分，我强调过从头思考，原因很简单，任何时候经验都是可能成为束缚的。就像从框架开发者的角度来说，放弃了灵活性，把自己局限在一定范围内简直是逆行倒施，但正是这样的局限才有可能在开发速度上和可维护性上带来质的飞升。

在这两年做框架开发的同时我也在做全栈教学的工作。这个过程中也发现对公司来说”授人以鱼”和“授人以渔“同样重要。因为无论教学做得多么成功，最后的产出物的质量仍然会受到受到学生的自身素质、工作内容等影响。特别是团队人员变化快时，教学的收益会特别低。而将能力服务化再提供给受众，可以抵御这种风险，因为服务自身可以不断沉淀、升级。后来在学习FBP时，与作者 J.P.Morrison 通信了解到 IBM 时代的 FBP 可视化工具的应用场景和这个项目非常像，而 FBP 当时在 IBM 内部取得了成功，他们甚至成功把全部可视化编辑的系统卖给了一家银行。这些信息也让我进一步意识到团队越大，构建上层建筑越有意义。在很多大公司里，光内部系统就有上百个，有大量复杂度在一定范围内的页面要开发，前端服务化的意义远大于我们站在自己固有的经验中所看到的程度。

到这里这一篇可以先告一段路了，之后组件库的碰到的常见问题和设计还有基于 web 的 IDE 通用架构会有另外的文章来说明。相比这些具体的技术实现，我更希望后面这些关于质变，以及如何形成质变的思考能带给读者更多收益。感谢阅读。

最后放出几张用户制作的页面:

<p align="center">
    <img src="https://zos.alipayobjects.com/rmsportal/UFPodqepPyaaIKEzcaYE.png" alt=""></img>
</p>

<p align="center">
    <img src="https://zos.alipayobjects.com/rmsportal/ZLZarjdupwqdayQoUNGV.png" alt=""></img>
</p>

<p align="center">
    <img src="https://zos.alipayobjects.com/rmsportal/ltxwmVOWHSOqChxmeGxy.png" alt=""></img>
</p>

<p align="center">
    <img src="https://zos.alipayobjects.com/rmsportal/sNfBuThtstskLoPbJyVR.png" alt=""></img>
</p>

### 答读者问

- 为什么社区从来没有流行过这样的东西？
    - 这个问题其实比较模糊，这和问”怎样做产品能成功”性质一样，我非常建议读者先读读纯银的文章，不管是技术还是产品都有收益。但是我仍然尝试回答一下。首先要做一套这样的开发环境设计到的技术栈太多，组件库、渲染引擎、IDE、分析引擎、后端服务每一个方面都要耗费相当大的人力。就算社区有这样的东西做出来了，也很少有团队有同等人力能拿去用，任何严肃地投入生产的项目都不可能拿一个自己掌握不住的工具去用的。其次，这样的东西就算有公司做出来并且在内部很流行，也很难为外界所知，因为这种集成开发环境首先需要大量内部系统的积累，在我们这里就是组件库、后端服务等。另外公司战略上的支持也是必不可少的。实际上据我了解，不管流不流行，每个大公司都有至少一套这样的系统。
- 框架部分为什么不用 redux ，现在的看起来像砍掉了 action 的 redux？
    - redux 本质上是个状态机，action 的设计能够约束变化来源，屏蔽来源细节，同时写代码时能把所有变化和数据本身写在一起，解决“数据在哪里，被怎么了”的问题。然而我们更倾向于向用户暴露更少的概念，让他用直觉来使用，由开发环境解决可维护性等问题。这其实是产品策略，和技术争论无关。
