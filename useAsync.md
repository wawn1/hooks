https://usehooks.com/

useCallback 如果asyncFunction对象变了，execute就新的对象，没有执行

useEffect在render之后执行，依赖就像函数声明参数，保证闭包里的值是最新的

提供一个返回promise的函数，promise里有异步任务，immediate控制是否立即执行，只能传true,false,字面量,此后一直不变

得到是否是任务执行中状态pending, 任务成功的结果value, 任务失败的原因error, execute控制是否执行

```js
const useAsync = (asyncFunction, immediate = true) => {
  const [pending, setPending] = useState(false);
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);

  // The execute function wraps asyncFunction and
  // handles setting state for pending, value, and error.
  // useCallback ensures the below useEffect is not called
  // on every render, but only if asyncFunction changes.
  const execute = useCallback(() => {
    setPending(true);
    setValue(null);
    setError(null);
    return asyncFunction()
      .then(response => setValue(response))
      .catch(error => setError(error))
      .finally(() => setPending(false));
  }, [asyncFunction]);

  // Call execute if we want to fire it right away.
  // Otherwise execute can be called later, such as
  // in an onClick handler.
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);

  return { execute, pending, value, error };
};
```

```js
function App() {
  const { execute, pending, value, error } = useAsync(myFunction, false);

  return (
    <div>
      {value && <div>{value}</div>}
      {error && <div>{error}</div>}
      <button onClick={execute} disabled={pending}>
        {!pending ? 'Click me' : 'Loading...'}
      </button>
    </div>
  );
}

// An async function for testing our hook.
// Will be successful 50% of the time.
const myFunction = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const rnd = Math.random() * 10;
      rnd <= 5
        ? resolve('Submitted successfully 🙌')
        : reject('Oh no there was an error 😞');
    }, 2000);
  });
};
```

