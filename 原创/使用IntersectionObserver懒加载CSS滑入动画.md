## CSS滑入动画
一开始我们不指定动画名称，当用户滑动到时再指定。
```CSS
.div {
  animation-duration: 0.5s;
  animation-fill-mode: both;
  animation-delay: 0.1s;
}
```
我们可以根据具体的需要（比如从左边滑入，或者从右边滑入），去修改 `translate3d`的第一个参数的正负号。
```CSS
@keyframes slideIn {
  0% {
    opacity: 0;
    transform: translate3d(18%, 0, 0);
  }

  100% {
    opacity: 1;
    transform: none;
  }
}
```
## 懒加载
使用 `IntersectionObserver` api，当我们的元素被用户观测到的时候，指定它的动画名称。
```tsx
 useEffect(() => {
    const intersectionObserver = new IntersectionObserver(function (
      entries
    ) {
      if (entries[0].isIntersecting === true) {
        ref.current!.style.animationName = `${Style.slidein}`;
      }
    });
    intersectionObserver.observe(ref.current!);
  }, []);
```