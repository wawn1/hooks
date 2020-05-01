https://github.com/streamich/react-use/blob/master/docs/useAudio.md

useAudio可以获取实时的audio的state

state包括duration时长，paused是否暂停，muted是否静音，time当前播放到的位置，volume音量0-1，buffered是TimeRanges对象

TimeRanges 对象表示用户的音视频缓冲范围。

缓冲范围指的是已缓冲音视频的时间范围。如果用户在音视频中跳跃播放，会得到多个缓冲范围。state里的buffered是parseTimeRanges处理后的数据结构

{start:number, end: number}[]

onProgress中会不断刷新buffered， onDurationChange第一次给buffered赋值

onprogress 事件在浏览器下载指定的视频/音频（audio/video）时触发，下载完就定了

```js
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
```

```js
const useAudio = createHTMLMediaHook('audio');

export default useAudio;
```



## Usage

```js
import React,{useState} from 'react'
import useAudio from './useAudio';

const srcs=['http://www.170mv.com/kw/antiserver.kuwo.cn/anti.s?rid=MUSIC_96804251&response=res&format=mp3|aac&type=convert_url&br=128kmp3&agent=iPhone&callback=getlink&jpcallback=getlink.mp3',
  'http://www.170mv.com/kw/antiserver.kuwo.cn/anti.s?rid=MUSIC_96765035&response=res&format=mp3|aac&type=convert_url&br=128kmp3&agent=iPhone&callback=getlink&jpcallback=getlink.mp3']

const App = () => {
  const [src, setSrc]=useState(srcs[0])
  const [audio, state, controls, ref] = useAudio({
    src,
    autoPlay: true,
  });

  return (
    <div>
      {audio}
      {src}
      <pre>{JSON.stringify(state, null, 2)}</pre>
      <button onClick={controls.pause}>Pause</button>
      <button onClick={controls.play}>Play</button>
      <br/>
      <button onClick={controls.mute}>Mute</button>
      <button onClick={controls.unmute}>Un-mute</button>
      <br/>
      <button onClick={() => controls.volume(.1)}>Volume: 10%</button>
      <button onClick={() => controls.volume(.5)}>Volume: 50%</button>
      <button onClick={() => controls.volume(1)}>Volume: 100%</button>
      <br/>
      <button onClick={() => controls.seek(state.time - 5)}>-5 sec</button>
      <button onClick={() => controls.seek(state.time + 5)}>+5 sec</button>
      <button onClick={() => setSrc(srcs[~~(Math.random()*2)])}>change Song</button>
    </div>
  );
};

export default App
```

## Reference

```js
const [audio, state, controls, ref] = useAudio(props);
const [audio, state, controls] = useAudio(<audio {...props}/>);
```

`audio` is React's `` element that you have to insert somewhere in your render tree, for example:

```jsx
<div>{audio}</div>
```

`state` tracks the state of the audio and has the following shape:

```json
{
  "buffered": [
    {
      "start": 0,
      "end": 425.952625
    }
  ],
  "time": 5.244996,
  "duration": 425.952625,
  "paused": false,
  "muted": false,
  "volume": 1
}
```

`controls` is a list collection of methods that allow you to control the playback of the audio, it has the following interface:

```ts
interface AudioControls {
  play: () => Promise<void> | void;
  pause: () => void;
  mute: () => void;
  unmute: () => void;
  volume: (volume: number) => void;
  seek: (time: number) => void;
}
```

`ref` is a React reference to HTML `` element, you can access the element by `ref.current`, note that it may be `null`.

And finally, `props` — all props that `` accepts.