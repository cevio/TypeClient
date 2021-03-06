---
sidebar: auto
---

# @typeclient/react

React驱动支持。推荐采用`FunctionComponent + hooks`模式编写应用。

## Hooks

新增基于路由级别的hooks

### useReactiveState

对于任意被`@vue/reactivity`包裹过的对象都将返回响应式的数据对象供react内部使用。

```tsx
import { reactive, ref } from '@vue/reactivity';
const state = reactive({ count: 0 });
const num = ref(100);
const { count, num } = useReactiveState(() => {
  return {
    count: state.count,
    num: num.value
  }
});
return <div>{count} - {num}</div>
```

> 也可以通过 `import { useReactiveState } from '@typeclient/react-effect'`获得。

### useContextState

返回请求级别ctx上数据的响应。

```tsx
@injectable()
class ABC {
  test() {
    const { count, status } = useContextState((ctx: Context) => {
      return {
        count: ctx.state.count,
        status: ctx.status.value
      }
    })
    return <div>{count} - {status}</div>
  }
}
```

### useApplicationContext

返回ctx对象

```tsx
const ctx = useApplicationContext() as Context;
```

### useContextEffect

路由生命周期，类似组件生命周期。它接受一个回调函数，表示路由`created`生命周期，如果返回一个回调函数，则表示路由`destroy`生命周期。

```tsx
useContextEffect(() => {
  console.log('router ready');
  return () => console.log('router destroyed');
})
```

### useComponent

为react提供一种新的组件模式

```tsx
import { Component, ComponentTransform } from '@typeclient/react';
@Component()
class ttt implements ComponentTransform {
  @inject(Abc) private Abc: Abc;
  public render(props: React.PropsWithoutRef<{}>) {
    return React.createElement('div', null, '123evio-' + this.Abc.abc());
  }
}
```

调用：

```tsx {10}
import { inject } from 'inversify';
import { Route, Controller, useMiddleware } from '@typeclient/core';
import { useComponent } from '@typeclient/react';
@Controller()
@useMiddleware(DemoMiddleware)
class router {
  @inject(ttt) private readonly ttt: ttt;
  @Route('/test')
  test() {
    const Cmp = useComponent(this.ttt);
    return <Cmp />
  }
}
```

### useSlot

它支持`Slot`插槽模式，我们可以通过`useSlot`获得`Provider`与`Consumer`。它是一种通过消息传送节点片段的模式。

```ts
const { Provider, Consumer } = useSlot(ctx.app);
<Provider name="foo">provider data</Provider>
<Consumer name="foo">placeholder</Consumer>
```

然后编写一个IOCComponent或者传统组件。

```tsx
// template.tsx
import { useSlot } from '@typeclient/react';
@Component()
class uxx implements ComponentTransform {
  render(props: any) {
    const { Consumer } = useSlot(props.ctx);
    return <div>
      <h2>title</h2>
      <Consumer name="foo" />
      {props.children}
    </div>
  }
}
```

最后在Controller上调用

```tsx
import { inject } from 'inversify';
import { Route, Controller } from '@typeclient/core';
import { useSlot } from '@typeclient/react';
import { uxx } from './template.tsx';
@Controller()
@Template(uxx)
class router {
  @inject(ttt) private readonly ttt: ttt;
  @Route('/test')
  test() {
    const { Provider } = useSlot(props.ctx);
    return <div>
      child ...
      <Provider name="foo">
        this is foo slot
      </Provider>
    </div>
  }
}
```

你能看到的结构如下:

```tsx
<div>
  <h2>title</h2>
  this is foo slot
  <div>child ...</div>
</div>
```