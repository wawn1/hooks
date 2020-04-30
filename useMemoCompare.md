来自https://usehooks.com/

useEffect产生异步函数任务，和setTimeot, dom类似，只不过执行时机不是微任务队列执行完成，而是在render之后

useEffect里面的函数引用的外部变量存到了闭包里

useMemoCompare, 接收一个对象和一个比较函数

如果比较函数返回true就返回旧的对象，否则返回接收的对象，也就是当前最新的对象

value来自哪里，value还是要重新计算一次得到，走一个计算流程，不会优化性能，只是保持原来对象引用不变

```js
function useMemoCompare(value, compare) {
  const previousRef = useRef();
  const previous = previousRef.current;

  const isEqual = compare(previous, value);

  useEffect(() => {
    if (!isEqual) {
      previousRef.current = value;
    }
  });

  return isEqual ? previous : value;
}
```

```js
const theObj = useMemoCompare(obj, prev => prev && prev.id === obj.id);
```

