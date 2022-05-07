## 效果图

![2022-05-07-20-48-14.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccef692b25fd4f4a867c455ce372124f~tplv-k3u1fbpfcp-watermark.image?)
## 最小可复现代码

html:
```html
    <div class="wrapper">
      <div class="typing-demo">This is a typing demo.</div>
    </div>
```

css:
```css
.wrapper {
  height: 100vh;
  // 居中
  display: grid;
  place-items: center;
}

.typing-demo {
  width: 22ch;
  animation: typing 2s steps(22), blink 0.5s step-end infinite alternate;
  white-space: nowrap;
  overflow: hidden;
  border-right: 3px solid;
  font-family: monospace;
  font-size: 2em;
}

@keyframes typing {
  from {
    width: 0;
  }
}

@keyframes blink {
  50% {
    border-color: transparent;
  }
}

```

## 原理

- ch：数字“0”的宽度，1ch 等于一个 0 的**宽度**，这里有22个字符，所以是22ch。
- border-right：用来模拟光标
- animation：css3动画。第一个动画名称为 `typing`，周期2s，`steps`方法将动画分段，这个动画用来打字的效果；第二个动画名称为 `blink`，用来实现光标的闪烁，`infinite`意为无限循环，`alternate`意为动画交替反向播放。
- font-size: monospace，因为使用`ch`需要等宽字体