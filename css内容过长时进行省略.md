# css:内容过长时进行省略

2023-4-4

今天在项目上提到了一个优化点:标题过长的时候进行省略处理.这时我猛然想起,前端的身份卡里还有切图仔的存在.依赖现成的组件插件太久,css都快忘了怎么写了.这边记录一下这个样式的写法

## 单行省略

```css
.title {
	width: 100px;
	text-overflow: ellipsis;
	overflow: hidden;
	white-space: nowrap; /* 让文字在一行内显示, 不换行 */
}
```

## 多行省略

下面这个写法在ie中可能会有点问题,但问题不算很大,同样是会把文字超出部分遮起来,只不过是不会显示省略号

```css
.title {
	width: 100px;
    word-break: break-all;
    text-overflow: ellipsis;
    overflow: hidden;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2; /* 这里是超出几行省略 */
}
```





