## React Hook
### 一、Hook 简介及使用
1. Hook 是一个特殊的函数，它可以让你“钩入” React 的特性。
2. 确保在**React函数组件**中才使用Hook，且是**顶层函数组件**使用，不能在嵌套函数及普通的javascript函数中使用。
3. 不要在循环，条件或嵌套函数中调用 Hook
### 二、为什么要使用Hook
1. 传统的Class组件在每次重新渲染时，都会创建一次实例，而函数组件只会创建一个实例，从而避免不必要的内存分配及检查。
2. 函数组件是无状态的，不存在state及生命周期，而使用Hook则可在函数组件中使用state及其它react特性。
3. 避免包装函数嵌套太深，无需使用mixins模式。
4. Class组件随着场景复杂，组件太大，维护成本高，无法将细化功能抽离出来。而使用hook则可**将逻辑抽离出来，达到多组件复用逻辑的功能**。
5. 在生命周期方法里订阅-注销事件是分开写在两个方法，代码较为分散，不利于管理，且易出现在订阅事件，却在组件销毁时未注销事件，如定时器这类，如果未注销，将会一直运行。
6. 简化组件的编写。
![示意图](https://img.php.cn/upload/image/937/347/231/1575366893854880.png)
### 三、[React 内置的 hook](https://react.docschina.org/docs/hooks-reference.html)
1. useState
2. useEffect
3. useLayoutEffect
4. useRef
5. useMemo
6. useContext
7. useReducer
8. useCallback
9. useImperativeHandle
10. useDebugValue

### 四、常用的hook
#### 1. [useState](https://react.docschina.org/docs/hooks-state.html)
- **什么时候要用state?**  
当某个变量或属性主动操作后发生改变，且页面元素需实时显示的，才需使用state，如果某个变量或属性的值是依赖其它state，而且我们不会主动去改变它的值时，此时没必要使用state.

- **怎么用？**  
```javascript
function app() {
    // 统计此函数调用次数，此处表现为：setThing每调用一次，此函数就会执行一次
    console.count('app ');
    // initState 可以为任何值，可为函数
    const initState = 0;
    const [thing, setThing] = useState(initState);
    // thing2的变化需显示，但它是的变化是由thing引起的，并不是主动操作引起的，所以无需State
    const thing2 = thing + 1;
    return (
    <div>
        <div>{thing}, {thing2}</div>
        <button onClick={() => setThing(thing + 1)}> + 1</button>
    </div>
    );
}
```
- **useReducer** 为useState的替代方案，机制与Redux类似。

#### 2. [useEffect](https://react.docschina.org/docs/hooks-reference.html#useeffect)
- **执行时机**  
effect将在浏览器绘制后延迟执行，但是会保证在新的渲染前执行。

- **怎么用**  
```javascript
function app() {
    const [thing, setThing] = useState(0);
    
    useEffect(() => {
        // todo     此副作用将在函数组件有变动时就会执行，相当于react生命周期中的 componentDidUpdate
    });
    
    // 副作用第二个参数为依赖项，当在副作用中使用外层变量时，必需放到依赖项中，此时相当于监听了相关属性的变化
    useEffect(() => {
        // todo     
        console.log(thing);
    }, [thing]);

    useEffect(() => {
        // todo     此副作用将在函数组件挂载时执行, 相当于react生命周期中的 componentDidMount
        // 副作用return回一个函数，此函数将在组件卸载时调用
        return () => {
            // todo     此副作用将在函数组件卸载时执行, 相当于react生命周期中的 componentWillUnMount
        };
    }, []);
}
```
- **useLayoutEffect**  
此副作用与useEffect结构一致，只是调用时机不一致。useLayoutEffect 是在浏览器绘制之前，将同步执行。

#### 3、[useRef](https://react.docschina.org/docs/hooks-reference.html#useref)
- useRef 返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。
- useRef 接收任意参数，此参数将作为其 .current 属性的初始值
- **用法**  
```javascript
function App() {
    const inputEl = useRef(null);
    const onButtonClick = () => {
        inputEl.current.focus();
    };

    const [thing, setThing] = useState(0);
    const ref = useRef(thing);
    // 监听thing的变化，且可拿到旧值
    useEffect(() => {
        console.log('old value: ', ref.current);
        console.log('new value: ', thing);
        ref.current = thing;
    }, [thing]);
    
    return (
        <div>
            <input ref={inputEl} type="text" placeholder="测试" />
            <button onClick={onButtonClick}>focus</button>
            <br/>
            <button onClick={() => setThing(thing + 1)}> + 1</button>
        </div>
    );
}
```  


#### 4、[useMemo](https://react.docschina.org/docs/hooks-reference.html#usememo)
- useMemo 返回一个 memoized 值，只有在它所依赖的属性发生改变时，这个值才会进行更新。
- **使用场景**  
使用在耗性能的场景中。前面useState有说，当函数组件中的变量或属性每发生一次改变，此函数就会执行一次。  
当组件中某一个变量是是需进行大量计算，耗性能的情况下，就应该使用useMemo来优化性能。
- **示例**  
```javascript
function App() {
    const [count, setCount] = useState(0);
    const [count2, setCount2] = useState(0);
    // count 或 count2 有变化时，此函数都会再次执行，所以count3会重新计算
    // 因循环大概需要1.2s，此时会明显看出结果会渲染的好慢
    const count3 = (() => {
        let v = 0;
        console.time('count3 time');
        // 此循环在我的机子大概需要1.2s
        for (let i = 0; i < 100000000; ++i) {
            v += Math.random() * i;
        }
        console.timeEnd('count3 time');
        return count + v;
    })();
    
    return (
        <div>
            count = {count}
            <br/>
            count2 = {count2}
            <br/>
            count3 = {count3}
            <br/>
            <button onClick={() => setCount(count + 1)}>count + 1</button>
            <button onClick={() => setCount2(count2 + 1)}>count2 + 1</button>
        </div>
    );
}
```
- **更换为使用useMemo**  
```javascript
function App() {
    const [count, setCount] = useState(0);
    const [count2, setCount2] = useState(0);
    // 更换为useMemo只有在组件挂载或count有改变时(非绝对的，当内存不够、久未使用时，可能会被释放)
    // count3才会被重新计算，所以当改变count2时，count3不会重新计算，页面不会显示好慢
    const count3 = useMemo(() => {
       let v = 0;
       console.time('count3 time');
       // 此循环在我的机子大概需要1.2s
       for (let i = 0; i < 100000000; ++i) {
           v += Math.random() * i;
       }
       console.timeEnd('count3 time');
       return count + v;
   }, [count]);
    
    return (
        <div>
            count = {count}
            <br/>
            count2 = {count2}
            <br/>
            count3 = {count3}
            <br/>
            <button onClick={() => setCount(count + 1)}>count + 1</button>
            <button onClick={() => setCount2(count2 + 1)}>count2 + 1</button>
        </div>
    );
}
```

### 五、自定义Hook  
 - 组件间需复用一些逻辑
 - 必须使用use开头
 - **示例**  
 ```javascript
function useCounter(initVal) {
    const [count, setCount] = useState(initVal);
    const increase = () => {
        setCount(count + 1);
    };
    const decrease = () => {
        setCount(count -1);
    };
    const reset = () => {
        setCount(initVal);
    };
    return {count, increase, decrease, reset};
}
function App() {
    const {count, increase, decrease, reset} = useCounter(0);
    const {count:count2, increase:increase2, decrease:decrease2, reset:reset2} = useCounter(10);
    return (
        <div>
            <div>
                count = {count}
                <div>
                    <button onClick={increase}>increase</button>
                    <button onClick={decrease}>decrease</button>
                    <button onClick={reset}>reset</button>
                </div>
            </div>
            <hr/>
            count2 = {count2}
            <div>
                <button onClick={increase2}>increase2</button>
                <button onClick={decrease2}>decrease2</button>
                <button onClick={reset2}>reset2</button>
            </div>
        </div>
    );
}
```

### 六、Hook注意事项  
在一函数组件中使用useState时，react将会在此组件节点上挂载一个state数组，当在此函数中调用useState时，将按useState调用顺序依次从这个state数组中取值，这样保证了在函数重新执行时，取到正确的state。  
这也是为什么不能在函数组件中根据条件来调用useState，其它Hook也是如此，如下：  
```javascript
function app() {
    const [thing, setThing] = useState(0);
    let thing2, setThing2;
    if (thing === 1) {
        [thing2, setThing2] = useState(0);
    }
}
```
如上代码会报如下错误：
* ##### React Hook "useState" is called conditionally. React Hooks must be called in the exact same order in every component render *

