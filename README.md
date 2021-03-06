# auto-css-sprite

## 其他语言版本
**python :** https://github.com/paper/auto-css-sprite-python

## Changelog
地址: [changelog.md](./changelog.md) 

内有示例图片 和 一些改动说明

## 预览图
![](demo/screen.png)

## 简要说明

#### 布局与设计

1. 整体页面简约为主，温和的色调，长时间使用，眼睛也不会疲劳
2. 左侧显示canvas图片和demo文本（会自适应），右侧是拖入图片区域，下面是一些操作按钮区域
3. “反转canvas颜色” 按钮，是为了让用户看得清比较偏白的图标

#### 使用流程

1. 拖一些图标进入“Drag img here” 区域时，底色会变，这时，你只要松手就可以了
2. 系统会自动生成合并的图片，还有demo代码
3. 由于浏览器限制，点击 “保存图片” 会新开窗口，然后在新开的窗口内，点击图片，会提示保存图片
4. 右下角 “设置” ，可以设置 “class标识” 和 “sprite路径”，让你的class命名区别于其他
5. class命名，采用 “acs” + “class标识” + “图片文件名” 。图片文件名任何不合法的字符都会被转化为 “-”。推荐“图片文件名”和“class标识”都是纯英文描述。比如：图片文件名 good.png、add.png 等等，class标识  一般是项目名称 projectName


## 算法

主要参考  http://www.aaai.org/Papers/ICAPS/2003/ICAPS03-029.pdf 算法

#### 基本思路
1. 排列矩形集合（S），按照高度从高到低进行排序
2. 取最高的1个矩形（s1）的宽高作为容器（Box）的初始宽高（BoxW，BoxH）
2. s1永远处于Box的左上角。BoxH 不变，BoxW 可以无线延伸
3. 判断第2个矩形（s2）能不能放到 Box 按照从左到右，从高到底这个顺序的空隙里面
	- 如果可以：放入 并 判断 BoxW 是否需要延伸
	- 如果不可以：放到s1的右边，BoxW 延伸
4. 继续第3步，判断s3, s4, s5......

#### 扩展思路
1. 根据Box的初始高度，可以得到最终最小面积（Nice）的第一个样本（Sample）
2. 通过阶梯增加Box的高度（boxHeightStep），得到多个样本
3. 比较每个样本的 面积利用率（U），取最高的那个

#### 算法细节
通过上面的描述，我们可以知道，算法的核心就是如何快速有效的判断 “Box 从左到右，从高到底这个顺序的空隙的坐标” 在哪？

我没有采用论文的切片算法（看得不是很懂），用的是自己研究的 “长条移动” 法。[ S里面最小高度是 MINH ]

1. 假想有一个宽度为1px宽度，高度为BoxH 的长条（L），覆盖在Box左上角上面
2. L 从左到右移动，x方向坐标（x），不断增加阶梯（xStep）
3. 分析 L 和 已经放入到Box的矩形（InS） 相交
	- 如果不相交，就把最新的s（sn） 放入到(x, 0)值的位置，更新 BoxW
	- 如果相交
		- 相交的矩形（xS）的高度总和（xSH）+ MINH >= H，说明不可能有位置放 sn 了，记录下当前x值（下次就不需要判断了），继续移动 L
		- 如果 xSH + MINH < H，有空隙。分析 L 与 xS 形成的 “相交线段” 空隙数量 和 长度
			- 如果所有空隙长度 分别小于 sn 高度（snH），继续移动 L
			- 如果有个空隙长度 大于 snH，放入 sn 在 （x, y）[y  是指空隙的y值]， 判断 sn 和 InS 有没有重叠，如果不重叠，放入。反之 继续移动 L
4. 判断下一个 sn ， 重复 1

#### 性能和精度
```
    // L 移动的步长
    // 【测试说明】以下测试，都是没有开启这2个开发工具。开启 firebug 或 chrome开发者工具，会影响测试值
    // 【测试环境】chrome35 和 firefox30，样本10个，boxHeightStep为16px
    // 【测试数据】big-and-small里面79张图片，单位 ms （毫秒）
    // xStep=1 chrome消耗的时间：1907, firefox消耗的时间：890, 最佳利用率：0.961
    // xStep=2 chrome消耗的时间：893 , firefox消耗的时间：506, 最佳利用率：0.955
    // xStep=4 chrome消耗的时间：517 , firefox消耗的时间：286, 最佳利用率：0.938
    var xStep = 2;
```

这是个NP问题，所以只能求近似最佳值。
影响这个算法性能和精度的地方有：

1. xStep的值
2. boxHeightStep 的值
3. Sample 个数

所以，如果你想要更高的精度（毕竟是本地程序，多等待那么几秒，问题不大）

请设置
```
    xStep = 1; Sample >= 20; boxHeightStep <= 16
```

#### 更多思考
1. 如何更加有效、智能的 设置 boxHeightStep 最佳值？
2. Sample 多少个才合理？
3. 有没有更好更快的算法？

## 结尾
写这个也是因为用过了[cssgaga](http://www.99css.com/archives/tag/cssgaga)，一个很棒的工具！作者推荐大家 [不要重复造轮子](http://www.99css.com/archives/977)，只要会用这个工具就可以了。

看过文章和评论后，研究一下文章给出的  [Optimal Rectangle Packing: Initial Results](http://www.aaai.org/Papers/ICAPS/2003/ICAPS03-029.pdf)，单独提取 [algo.js](./js/algo.js) 核心算法 ，内有非常详细的注释。

我已经写了一个简单的[python](https://github.com/paper/auto-css-sprite-python)版本

大家可以随意改成 java ，php，c/c++ 等其他版本。 :D







