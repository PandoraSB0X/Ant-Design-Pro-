# 记Ant Design Pro

# Ant Design Pro

> [官方文档](https://pro.ant.design/docs/getting-started-cn)
>
> [网友总结参考较详细](https://www.codercto.com/a/26106.html)

## 安装（搭建脚手架）

从GitHub仓库中直接安装最新的脚手架代码

```
$ git clone --depth=1 https://github.com/ant-design/ant-design-pro.git my-project
$ cd my-project
```

生成完整的框架，目录结构如下

```
├── config                   # umi 配置，包含路由，构建等配置-----相当于改版之前的common
├── mock                     # 本地模拟数据
├── public
│   └── favicon.png          # Favicon
├── src
│   ├── assets               # 本地静态资源
│   ├── components           # 业务通用组件
│   ├── e2e                  # 集成测试用例
│   ├── layouts              # 通用布局
│   ├── models               # 全局 dva model
│   ├── pages                # 业务页面入口和常用模板------即改版前的routes
│   ├── services             # 后台接口服务
│   ├── utils                # 工具库
│   ├── locales              # 国际化资源
│   ├── global.less          # 全局样式
│   └── global.js            # 全局 JS
├── tests                    # 测试工具
├── README.md
└── package.json
```

## 本地开发

安装依赖包

```
$ npm install
```

依赖包安装完毕之后，开始运行

```
$ npm start
```

启动完成后会自动打开浏览器访问 [http://localhost:8000](http://localhost:8000/)

# 权限管理的概念

> 基于角色的访问控制方法（Role-Based-Access Control，简称RBAC）是目前公认的解决大型企业的统一资源访问控制的有效方法：减少授权管理的复杂性，降低管理开销，灵活的支持企业的安全策略，并对企业的变化有很大的伸缩性。

**RBAC-0是基于角色的访问控制方法的最基础的模型，在原来的“用户-权限”结构中变成 ”用户-角色-权限“结构**

### 用户管理

提供用户账户的创建，编辑，启用，停用，删除的功能，具体依赖于邮箱，手机号或其他信息创建账号，根据公司实际情况而定。

### 角色管理

角色管理包含了用户&角色的关联，角色&权限的关联关联，主要包括：

- 角色的创建、编辑、删除的功能
- 角色下用户的添加和删除
- 角色权限的配置功能（配置该角色具备什么样的权限）

这种结构中，创建不同的角色，将权限直接赋予角色，用户只需要关联角色就可以获得该角色所对应的权限。

- 这样不用每个用户都设置一遍权限，只需要设置角色的权限
- 用户角色变化的时候，比如从技术部转到运营部，只需要将用户的角色变更，相应的权限就会跟着变更

### 功能权限

> 定义人员是否可以访问某个页面

- **菜单级别的权限：可见性**：如果没有权限，则不展示菜单

- 实现：

  ``` js
  //router.config.js   
  //选取部分为例
  // app
    {
      path: '/',
      component: '../layouts/BasicLayout',
      Routes: ['src/pages/Authorized'],
      authority: ['admin', 'user','tourist'],//权限控制(数组中的为有权限进入首页的人员)
      routes: [
        // dashboard
        { path: '/', redirect: '/dashboard/analysis' },
        {
          path: '/dashboard',
          name: 'dashboard',
          icon: 'dashboard',
          authority: ['admin', 'user'],//权限控制(数组中的为对dashboard可见，tourist此时进入页面不可见dashboard菜单)
          routes: [
            {
              path: '/dashboard/analysis',
              name: 'analysis',
              component: './Dashboard/Analysis',
            }
          ],
        },
  ```

  ``` js
  //mock/user.js(用户登录的接口及数据)
  //可以控制登录用户的用户名
  'POST /api/login/account': (req, res) => {
      const { password, userName, type } = req.body;
      if (password === 'ant.design' && userName === 'admin') {
        res.send({
          status: 'ok',
          type,
          currentAuthority: 'admin',
        });
        return;
      }
      if (password === 'ant.design' && userName === 'user') {
        res.send({
          status: 'ok',
          type,
          currentAuthority: 'user',
        });
        return;
      }
      if (password === 'ant.design' && userName === 'tourist') {
        res.send({
          status: 'ok',
          type,
          currentAuthority: 'tourist',
        });
        return;
      }
      res.send({
        status: 'error',
        type,
        currentAuthority: 'guest',
      });
    },
  ```

- 总结：菜单栏其实就是每一个路由。因此可以在路由中控制菜单栏的权限。

### 数据权限

> 定义人员在页面中，可以查看的数据的范围（后端会返回数据）

- 比如，查看销售记录的功能，用户A,B都有权限使用，但是A有权看到公司所有人的，B只能看到他自己的。这就涉及数据权限的控制了。
- 数据权限依赖于功能权限，是对功能权限的进一步描述，说明角色在指定的功能点上的数据控制权限。
- 对于数据权限来说，可以根据需要设置多个数据类型字段。（数据类型就是数据中的某一个字段，数据对象即数据类型的值）

### 按钮/字段权限

> 定义人员可以操作的数据，可以查看的字段

- **页面元素级别权限：可操作性**：如果没有该权限，操作时报错，比如点击时提示“对不起，您没有该操作权限”

**分为部门的增删改查与用户的增删改查，配置权限，这样就能加强扩展性，比如取消或者合并某个部门的时候**

### 基于Ant Design Pro2的权限管理（未采用，供备用参考）

> 目标：权限的维度希望能够控制到按钮层，包括菜单、路由

antd的方案基于角色维度来进行。config设定当前路由的准入角色。权限组建接受当前角色信息，如此判断渲染的内容。

```
如需对某些页面进行权限控制，只须在路由配置文件 router.config.js 中设置 authority 属性即可，代表该路由的准入权限，pro 的路由系统中会默认包裹 Authorized 进行判断处理。
```

缺陷：umi2中基于约定路由的形式。在路由初始化时不能动态配置config。及不能动态修改authority字段。而且基于角色的权限不能满足我们的需求。我们希望精确到用户的权限配置。后端下发菜单，路由以及按钮list。

## 实现

为解决该问题。我们在router.config.js新增code字段。标记路由的唯一code。Routes定义路由层的父级组件。在该组件中进行路由拦截，实现路由级别权限控制。 // config.js

```
[
  { path: '/', component: './pages/index.js' },
  { path: '/list', component: './pages/list.js',code:123, Routes: ['./routes/PrivateRoute.js'] },
]

作者：en1475230123000
链接：https://juejin.im/post/5baeedbe5188255c4834c19d
```

```
// PrivateRoute.js 
if (route.code && menuList && menuList.indexOf(route.code) == -1) {
    router.replace('/exception/403');
  }
```

### 菜单权限

基于pro的菜单方案进行了修改。直接动刀BasicLayout.js,修改原先基于authority字段的菜单逻辑。改至code字段。

### 新增路由管理

路由表由前端提供，路由名称，（包括路由，菜单，以及按钮类型）。后端存至服务端。再有对应的token下发对应的code list。

### 按钮

写个高阶组件封装下button。 over



# icon的更换与增加

> 直接在ant图标里获取type值，然后添加 icon：‘iconName’，或者下载阿里图标生成静态文件（）之后再补充



# 解读Ant Design Form中的onChange

> 转载请说明出处 [点我达技术](http://tech.dianwoda.com/2018/03/22/ant-form/)

主要讲[Ant Design Form](https://ant.design/components/form-cn/)组件使用中碰到的问题（onChange），顺便源码解析。

接下来说的Form代表ant-form([react-component/form](https://github.com/react-component/form))

## 一、Form的主要作用

首先了解什么是Form

> React High Order Form Component(web & react-native)

1、是一个Form高阶组件，[HOC](https://reactjs.org/docs/higher-order-components.html)官方文档已经说了很详细了，简单说下，`HOC`是设计模式中`装饰模式`的一个实践，在不改变原有的用途上进行组件增强。

2、让被包装的组件具备表单功能，其中的表单组件具备数据双向绑定，以及一些校验等一系列功能。

## 二、创建Form的大致原理（如何绑定）

下面是最基本Form代码片段，里面有一个id/key为`name`的输入框。

```
class Demo1 extends Component {
  render() {
    const { getFieldProps } = this.props.form;
    return (
      <div>
        <Input {...getFieldProps('name')} />
      </div>
    );
  }
}
export default Form.create()(Demo1);
```

通过`Form.create`初始化这个组件；`HOC`对被包装的`props`进行了拦截，注入了自己的对象`form`；`getFieldProps`基本等价于`getFieldDecorator`只是写法不同，这个方法主要返回`onChange(onXXX) value`这两个重要的双向绑定的属性，最终返回值作用于`Input`上面

再通过`form.validateFields/validateFieldsAndScroll`就能完成简单的数据提交了。

下面是通过react-dev-tools在Chrome中查看Input的属性结果

[![input-props](https://raw.githubusercontent.com/zhongjie-chen/antd-form-demo/master/imgs/1.png)](https://raw.githubusercontent.com/zhongjie-chen/antd-form-demo/master/imgs/1.png)

其它属性是`ant Input`的属性。

通过上幅图我们大致了解了`getFieldProps`主要返回了哪些值。

## 三、Form中的onChange（如何存储，如何更新）

**\*我看到上图中的的onChange方法，是一个名叫onCollect被bind后的方法，我们搜索源码中的onCollect***

```
// 代码片段一
onCollect(name_, action, ...args) {
  const { name, field, fieldMeta } = this.onCollectCommon(name_, action, args);
  const { validate } = fieldMeta;
  const newField = {
    ...field,
    dirty: hasRules(validate),
  };
  // setFields请查看代码片段二
  this.setFields({
    [name]: newField,
  });
},
```

通过`onCollectCommon`方法 最终返回新的`newField`然后调用`setFields`方法

```
// 代码片段二
setFields(maybeNestedFields) {
  const fields = this.fieldsStore.flattenRegisteredFields(maybeNestedFields);
  this.fieldsStore.setFields(fields);
  if (onFieldsChange) {
    const changedFields = Object.keys(fields)
      .reduce((acc, name) => set(acc, name, this.fieldsStore.getField(name)), {});
    onFieldsChange(this.props, changedFields, this.fieldsStore.getNestedAllFields());
  }
  this.forceUpdate();
},
```

1、看`代码片段二`可得最终表单里的数据是放到一个叫`fieldsStore`里。

2、通过`fieldsStore`把新的`Fields`设置到这个对象里，再通过`this.forceUpdate()`手动去做`render`。*这句可以忽视（解析Element->Dom Element->virtual Dom diff old virtual Dom 生成新的页面）。*

## 四、被包装组件中的onChange

> 有一个需求，一个渠道的多选控件，有选项【点我达】【点我吧】【饿了么】还有一个【全部】，全部跟其它选择具有互斥效果。比如选了【点我达】，【全部】这个选择要删除掉；比如选了【全部】，其他的选项要清空掉。

如下图所示

[![selector](https://raw.githubusercontent.com/zhongjie-chen/antd-form-demo/master/imgs/select.gif)](https://raw.githubusercontent.com/zhongjie-chen/antd-form-demo/master/imgs/select.gif)

下面是最开始实现的代码

```
// 代码片段三
const data = [
  { name: '全部', value: 0 },
  { name: '点我达', value: 1 },
  { name: '点我吧', value: 2 },
  { name: '饿了么', value: 3 },
]
class Demo2 extends Component {
  render() {
    const { getFieldProps, setFieldsValue, getFieldValue } = this.props.form;
    return (
      <div>
        <Select
          {...getFieldProps('name', {
          })}
          style={{ width: 800 }}
          mode="multiple"
          onChange={(nextValues) => {
            let targetValues = []
            const nowValues = getFieldValue('name') || [];
            if (nowValues.length > nextValues.length) {
              targetValues = nextValues;
            } else {
              const selectValue = nextValues.find(nv => nowValues.indexOf(nv) === -1)
              if (selectValue === '0') {
                targetValues = ['0']
              } else {
                targetValues = nextValues.filter(x => x !== '0');
              }
            }
            setFieldsValue({ name: targetValues })
          }}
        >
          {data.map((item) => <Option key={item.value}>{item.name}</Option>)}
        </Select>
      </div>
    );
  }
}
```

监听`onChange`的变化，然后根据变化的数据转化成业务需要的数据，通过`setFieldsValue`设置的新的数据。但是很遗憾更新值是失败的。

在上面的基础上，改下代码

```
  // 代码片段四
  // some code...
  setTimeout(() => {
    setFieldsValue({ name: targetValues })
  }, 0);
  // some code...
```

把`setFieldsValue`放到下个事件循环中去执行，才能成功。这写法虽然实现了功能，但是有两缺点：

1、`setTimeout 0`代码不雅观。

2、本来一次渲染就解决的问题，现在要两次渲染。

## 五、为什么在onChange中去setFieldsValue是没有效果的呢？

我们带着这个疑问来看源码，开始讲的`onCollect`是收集表单组件的变化，所以手动写的`onChange`方法，是不会直接作用于原始的方法上；`onChange`方法实际会在`onCollect`中去执行；接下来看下源码具体怎么执行的，

`代码片段一`可以看出`onCollect`方法中调用了`onCollectCommon`，根据意思是通用的收集变化的处理方法，看该方法源码

```
// action实际上是trigger 默认是onChange
onCollectCommon(name, action, args) {
  const fieldMeta = this.fieldsStore.getFieldMeta(name);
  if (fieldMeta[action]) {
    // onchange getFieldProps中写法
    fieldMeta[action](...args);
  } else if (fieldMeta.originalProps && fieldMeta.originalProps[action]) {
    // onchange getFieldDecorator中的写法
    fieldMeta.originalProps[action](...args);
  }
  // some code...
  // onchange同步执行完后再执下面代码，下面的返回并不会受onchange同步执行的影响
  return ({ name, field: { ...field, value, touched: true }, fieldMeta });
},
```

`onChange`之类的方法是在`fieldMeta[action](...args);`或者`fieldMeta.originalProps[action](...args);`这两行代码执行的，在这进行`setFieldsValue`（代码片段三中执行的方法）会进行一次刷新，但是后续执行`setFields`会覆盖掉之前的数据，`setFields`并不会受中间的`setFieldsValue`影响，还是设置原来本需要设置的值。所以就很好解释了`为什么在onChange中去setFieldsValue是没有效果的呢？`。

## 六、Form没有提供这样的API吗

仔细查阅文档后发现有一个属性，`options.normalize`

> 官方解释：转换默认的 value 给控件；function(value, prevValue, allValues): any

```
class Demo4 extends Component {
  render() {
    const { getFieldProps } = this.props.form;
    return (
      <div>
        <Select
          {...getFieldProps('name', {
            normalize: (value, prevValue, allValues) => {
              let targetValues = []
              const nowValues = prevValue || [];
              if (nowValues.length > value.length) {
                targetValues = value;
              } else {
                const selectValue = value.find(nv => nowValues.indexOf(nv) === -1)
                if (selectValue === '0') {
                  targetValues = ['0']
                } else {
                  targetValues = value.filter(x => x !== '0');
                }
              }
              return targetValues;
            }
          })}
          style={{ width: 800 }}
          mode="multiple"
        >
          {data.map((item) => <Option key={item.value}>{item.name}</Option>)}
        </Select>
      </div>
    );
  }
}
```

以上是`normalize`的代码，通过转换返回新的值，不会出现两次渲染。接下来看下源码是如何实现的。

在代码片段2中`this.fieldsStore.setFields(fields)`，查看`createFieldsStore`文件中的`setFields`方法

```
setFields(fields) {
  const fieldsMeta = this.fieldsMeta;
  const nowFields = {
    ...this.fields,
    ...fields,
  };
  const nowValues = {};
  Object.keys(fieldsMeta)
    .forEach((f) => nowValues[f] = this.getValueFromFields(f, nowFields));
  Object.keys(nowValues).forEach((f) => {
    const value = nowValues[f];
    const fieldMeta = this.getFieldMeta(f);
    // 这里写的很明白 存在normalize 调用这个方法 返回新的nowFields
    if (fieldMeta && fieldMeta.normalize) {
      const nowValue =
              fieldMeta.normalize(value, this.getValueFromFields(f, this.fields), nowValues);
      if (nowValue !== value) {
        nowFields[f] = {
          ...nowFields[f],
          value: nowValue,
        };
      }
    }
  });
  this.fields = nowFields;
}
```

`normalize`这个是官方提供的在改变数据，重新渲染之前，提供的一个转换`fields`的方法。

### 七、normalize存在的问题？

如果表单组件使用了`rules`检验，每次值改变`normalize`会调用两次，看源码如果是具有检验的会去绑定`onCollectValidate`,后续调用`validateFieldsInternal`

```
validateFieldsInternal(fields, {
  fieldNames,
  action,
  options = {},
}, callback) {
  //some code...
  // 这里会执行一次
  this.setFields(allFields);
  // some code...
  
    // 这里会执行第二次
    this.setFields(nowAllFields);
    // some code...
},
```

1、上面精简了代码，提取两个关键的点，两次的`setFields`，第一次是正常的数据更新(dirty: true)，第二次是检验后会产生了一些新的数据（是否检验成功等信息）再次去更新渲染(dirty: false)。

2、因为`setFields`中会执行`normalize`，这也是会执行两次的原因。

3、个人理解第一次执行去做`normalize`就可以，第二次就没必要去回调了。（有不同看法可以探讨）

## 八、总结

1、不要在onChange中去设置Form中的值，要在`normalize`这个属性中去做。

2、使用中碰到比较费解问题，所以需要看源码来理解。



# ant pro实现获取下拉列表，图片上传

> [参考地址](https://blog.csdn.net/qq_33514421/article/details/81507354)

