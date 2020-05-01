# `useSetState`

React state hook that creates `setState` method which works similar to how `this.setState` works in class components—it merges object changes into current state.

和class的setState一样，可以传函数和对象，使用合并覆盖来更新

```js
import React, { useState, useCallback } from 'react';

const useSetState = (initialState) => {
  const [state, set] = useState(initialState);
  const setState = useCallback(
    (patch) => {
      set((prevState) => Object.assign({}, prevState, patch instanceof Function ? patch(prevState) : patch));
    },
    [set]
  );

  return [state, setState];
};

```



## Usage

```js
import {useSetState} from 'react-use';

const Demo = () => {
  const [state, setState] = useSetState({});

  return (
    <div>
      <pre>{JSON.stringify(state, null, 2)}</pre>
      <button onClick={() => setState({hello: 'world'})}>hello</button>
      <button onClick={() => setState({foo: 'bar'})}>foo</button>
      <button 
        onClick={() => {
          setState((prevState) => ({
            count: (prevState.count || 0) + 1,
          }))
        }}
      >
        count
      </button>
    </div>
  );
};
```

## Reference

```js
const [state, setState] = useSetState({cnt: 0});

setState({cnt: state.cnt + 1});
setState((prevState) => ({
  cnt: prevState + 1,
}));
```