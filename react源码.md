# React源码

## 数据结构

### element对象

```jsx
import { jsx as _jsx } from 'react/jsx-runtime'
import { jsxs as _jsxs } from 'react/jsx-runtime'

function App() {
  // /*#__PURE__*/ 注释告诉压缩工具 _jsxs 是一个纯函数调用，如果它的结果未被使用，可以安全地删除。
  return /*#__PURE__*/_jsxs("div", {
    id: "div",
    class: "div",
    children: [/*#__PURE__*/_jsx("span", {}), /*#__PURE__*/_jsx("p", {
      id: "p"
    })]
  });
}
```

babel将jsx代码编译成js代码，通过jsx接收两个参数，第一个参数 `type` 为组件的 type，第二个参数是其他配置，可能有第三个参数为组件的 `children`，返回一个 `ReactElement` 数据结构。

```tsx
export const jsx = (type: ElementType, config: any, ...children: any) => {
    let key: Key = null;
    let ref: Ref = null;
    const props: Props = {};
    const childrenLength = children.length;
    
    // 将key和ref单独存储，剩余的都存储在props中
    for (const prop in config) {
        const val = config[prop];
        if (prop === 'key') {
            if (val !== undefined) {
                key = '' + val;
            }
            continue;
        }
        if (prop === 'ref') {
            if (val !== undefined) {
                ref = val;
            }
            continue;
        }
        if ({}.hasOwnProperty.call(config, prop)) {
            props[prop] = val;
        }
    }
    
    if (childrenLength) {
        if (childrenLength === 1) {
            props.children = children[0];
        } else {
            props.children = children;
        }
    }
    return ReactElement(type, key, ref, props);
};


const ReactElement = function ( type, key, ref, props) {
    const element = {
        $$typeof: REACT_ELEMENT_TYPE,
        type,
        key,
        ref,
        props,
    };
    return element;
};
```

### fiber对象

对react执行过程中元素状态的描述

```JavaScript
const fiber = {
  // 标识
  tag, // Fiber 的类型（如 FunctionComponent、ClassComponent、HostComponent 等）
  key, // 唯一标识，用于区分同级元素
  elementType, // 元素的类型（如组件函数或类，或原生 DOM 元素类型）
  type, // 与 elementType 类似，但在某些情况下可能不同（如使用 React.lazy 时）
  stateNode, // 与 Fiber 关联的本地状态（如组件实例或 DOM 元素）

  // 链接
  return, // 指向父 Fiber
  child, // 指向第一个子 Fiber
  sibling, // 指向下一个兄弟 Fiber
  index, // 当前 Fiber 在兄弟 Fiber 中的索引

  // 更新
  pendingProps, // 新的 props，在更新期间使用
  memoizedProps, // 上一次渲染时的 props
  updateQueue, // 状态更新队列
  memoizedState, // 上一次渲染时的 state

  // 副作用
  effectTag, // 副作用标记，指示需要执行的操作（如更新、删除等）
  nextEffect, // 指向下一个需要处理副作用的 Fiber
  firstEffect, // 指向第一个副作用 Fiber
  lastEffect, // 指向最后一个副作用 Fiber

  // 优先级
  expirationTime, // 当前 Fiber 的过期时间，用于调度优先级
  childExpirationTime, // 子 Fiber 的过期时间

  // 其他
  alternate, // 当前 Fiber 的替代 Fiber（用于双缓冲）
  // 其他内部属性
};
```

worktag是对元素类型的进一步抽象

```JavaScript
export type WorkTag =
    | typeof FunctionComponent
    | typeof HostRoot
    | typeof HostComponent
    | typeof HostText;

export const FunctionComponent = 0;
export const HostRoot = 3;   // hsotroot代表生成的中间空节点

export const HostComponent = 5;   // 原生节点  div span等
// <div>123</div>
export const HostText = 6;
```



