https://github.com/streamich/react-use/blob/master/docs/useCookie.md

根据cookie名称拿到最新cookie的value，提供update, delete方法

```js
import { useState, useCallback } from 'react';
import Cookies from 'js-cookie';

const useCookie = (cookieName) => {
  const [value, setValue] = useState(() => Cookies.get(cookieName) || null);

  const updateCookie = useCallback(
    (newValue, options) => {
      Cookies.set(cookieName, newValue, options);
      setValue(newValue);
    },
    [cookieName]
  );

  const deleteCookie = useCallback(() => {
    Cookies.remove(cookieName);
    setValue(null);
  }, [cookieName]);

  return [value, updateCookie, deleteCookie];
};

```

## Usage

```js
import { useCookie } from "react-use";

const Demo = () => {
  const [value, updateCookie, deleteCookie] = useCookie("my-cookie");
  const [counter, setCounter] = useState(1);

    //每次加载组件第一次执行，删除my-cookie
  useEffect(() => {
    deleteCookie();
  }, []);
// 将my-cookie的值更新为my-awesome-cookie-${counter}
  const updateCookieHandler = () => {
    updateCookie(`my-awesome-cookie-${counter}`);
    setCounter(c => c + 1);
  };

  return (
    <div>
      <p>Value: {value}</p>
      <button onClick={updateCookieHandler}>Update Cookie</button>
      <br />
      <button onClick={deleteCookie}>Delete Cookie</button>
    </div>
  );
};
```