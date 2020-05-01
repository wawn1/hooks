函数式组件里，不是useState,useEffect，每次setState导致重新render都会从上到下执行一遍，这里createElement或者cloneElement会执行很多遍，为什么音乐是继续播放，而不是从头开始呢？

createElement只是更新了dom节点属性，对应节点的currentTime，真实dom的其他属性不会刷新。tree diff比对更新的。

console.log(ref.current **&&** ref.current.currentTime)打印一直向前，没有每次变0

其实用react.createElement和写标签是一样的<audio ...props ref={ref} on...>

每次create只是js内存虚拟dom, 真实dom没有重新create的

useMemo也不能阻止计算，它是根据依赖返回新对象还是旧的对象，其实新对象已经计算产生了，只不过返回旧的对象而已，

useCallback不一样，不需要计算，节省不了性能，确保返回的是旧的函数对象

也不需要阻止计算，tree diff 就需要新的虚拟dom，不然跟谁比，新旧虚拟dom树，create就是新的dom树节点

src 发生变化的时候触发onDurationChange， onTimeUpdate， onProgress

更新了duration， time， buffered

还有paused， muted， volume  在useEffect里面继承上次歌曲的设置

```js
import React, {useState, useEffect, useRef} from 'react'
import useSetState from './useSetState';

const parseTimeRanges = (ranges) => {
    const result = []; // {start:number, end: number}[]
  
    for (let i = 0; i < ranges.length; i++) {
      result.push({
        start: ranges.start(i),
        end: ranges.end(i),
      });
    }
  
    return result;
  };

  const createHTMLMediaHook = (tag) => {
    const useHook = ( elOrProps ) => {
      
      const [state, setState] =  useSetState({
        buffered: [],
        time: 0,
        duration: 0,
        paused: true,
        muted: false,
        volume: 1,
      });
      
      // 先自定义的事件用来获取实时的值到state,用户的事件最后必须处理
      const wrapEvent = (userEvent, proxyEvent) => {
        return event => {
          try {
            proxyEvent && proxyEvent(event);
          } finally {
            userEvent && userEvent(event);
          }
        };
      };
      // 监听事件来获取实时数据放入state
      const onPlay = () => setState({ paused: false });
      const onPause = () => setState({ paused: true });
      const onVolumeChange = () => {
        const el = ref.current;
        if (!el) {
          return;
        }
        setState({
          muted: el.muted,
          volume: el.volume,
        });
      };
      const onDurationChange = () => {
        const el = ref.current;
        if (!el) {
          return;
        }
        const { duration, buffered } = el;
        setState({
          duration,
          buffered: parseTimeRanges(buffered),
        });
      };
      const onTimeUpdate = () => {
        const el = ref.current;
        if (!el) {
          return;
        }
        setState({ time: el.currentTime });
      };
      const onProgress = () => {
        const el = ref.current;
        if (!el) {
          return;
        }
        setState({ buffered: parseTimeRanges(el.buffered) });
      };
      // 可以传入react节点或者，单纯传属性需要创建节点(方便)，并绑定ref
      const ref = useRef(null);
      let element;
      let props;
      
      if (React.isValidElement(elOrProps)) {
        element = elOrProps;
        props = element.props;
      } else {
        props = elOrProps
      }
      if (element) {
          console.log('clone')
        element = React.cloneElement(element, {
          controls: false,
          ...props,  // 包含了src
          ref,
          onPlay: wrapEvent(props.onPlay, onPlay),
          onPause: wrapEvent(props.onPause, onPause),
          onVolumeChange: wrapEvent(props.onVolumeChange, onVolumeChange),
          onDurationChange: wrapEvent(props.onDurationChange, onDurationChange),
          onTimeUpdate: wrapEvent(props.onTimeUpdate, onTimeUpdate),
          onProgress: wrapEvent(props.onProgress, onProgress),
        });
      } else {
        console.log('createElement')
        element = React.createElement(tag, {
          controls: false,
          ...props, // 包含了src
          ref,
          onPlay: wrapEvent(props.onPlay, onPlay),
          onPause: wrapEvent(props.onPause, onPause),
          onVolumeChange: wrapEvent(props.onVolumeChange, onVolumeChange),
          onDurationChange: wrapEvent(props.onDurationChange, onDurationChange),
          onTimeUpdate: wrapEvent(props.onTimeUpdate, onTimeUpdate),
          onProgress: wrapEvent(props.onProgress, onProgress),
        } ); 
      }
        console.log(ref.current && ref.current.currentTime)
  
      // Some browsers return `Promise` on `.play()` and may throw errors
      // if one tries to execute another `.play()` or `.pause()` while that
      // promise is resolving. So we prevent that with this lock.
      // See: https://bugs.chromium.org/p/chromium/issues/detail?id=593273
      let lockPlay = false;
      // 通过ref来控制播放，暂停，跳跃播放，音量，是否静音
      const controls = {
        play: () => {
          const el = ref.current;
          if (!el) {
            return undefined;
          }
  
          if (!lockPlay) {
            const promise = el.play();
            const isPromise = typeof promise === 'object';
  
            if (isPromise) {
              lockPlay = true;
              const resetLock = () => {
                lockPlay = false;
              };
              promise.then(resetLock, resetLock);
            }
  
            return promise;
          }
          return undefined;
        },
        pause: () => {
          const el = ref.current;
          if (el && !lockPlay) {
            return el.pause();
          }
        },
        seek: (time) => {
          const el = ref.current;
          if (!el || state.duration === undefined) {
            return;
          }
          time = Math.min(state.duration, Math.max(0, time));
          el.currentTime = time;
        },
        volume: (volume) => {
          const el = ref.current;
          if (!el) {
            return;
          }
          volume = Math.min(1, Math.max(0, volume));
          el.volume = volume;
          setState({ volume });
        },
        mute: () => {
          const el = ref.current;
          if (!el) {
            return;
          }
          el.muted = true;
        },
        unmute: () => {
          const el = ref.current;
          if (!el) {
            return;
          }
          el.muted = false;
        },
      };
      // 如果src变化，使用ref重新开始，第一次render之后，如果设置自动播放，就自动播放
      useEffect(() => {
        const el = ref.current;
          // ref的元素不存在的，错误处理  第一次render之后ref.current才不是null,前面都是null，有时候也可能为null
        if (!el) {
          if (process.env.NODE_ENV !== 'production') {
            if (tag === 'audio') {
              console.error(
                'useAudio() ref to <audio> element is empty at mount. ' +
                  'It seem you have not rendered the audio element, which it ' +
                  'returns as the first argument const [audio] = useAudio(...).'
              );
            } else if (tag === 'video') {
              console.error(
                'useVideo() ref to <video> element is empty at mount. ' +
                  'It seem you have not rendered the video element, which it ' +
                  'returns as the first argument const [video] = useVideo(...).'
              );
            }
          }
          return;
        }
  
        setState({
          volume: el.volume,
          muted: el.muted,
          paused: el.paused,
        });
  
        // Start media, if autoPlay requested.
        if (props.autoPlay && el.paused) {
          controls.play();
        }
      }, [props.src]);
  
      return [element, state, controls, ref];
    };
  
    return useHook;
  };

const useAudio = createHTMLMediaHook('audio');

export default useAudio;
```

