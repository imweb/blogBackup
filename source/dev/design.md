title: IMWEB团队开发规范——视觉规范
date: 2014-11-04 19:42:47
comments: true
author: Kael
about: https://github.com/yorts52
---
<ul class="dev-guide-box">
    <li><a href="/dev/index.html">1) 脚手架</a></li><li><a href="/dev/design.html">2) 视觉规范</a></li><li><a href="/dev/html.html">3) HTML规范</a></li><li><a href="/dev/js.html">4) Javascript规范</a></li><li><a href="/dev/css.html">5) CSS规范</a></li><li><a href="/dev/other.html">6) 其他规范</a></li>     
</ul>

#二、视觉规范

|优先级|需要check的项目|产品/交互/视觉|备注|
|:-----|:--------------|:-------------|:---|
|A　　　|是否有视觉规范|需提供||
|A　　　|是否有交互稿|需提供|需求评审必须 `交互评审通过，产品抄送开发侧`、`交互有变更，更新后的交互稿抄送给开发侧`|
|B　　　|任何按钮的三态：`normal`，`hover`，`active`|视觉交付时一并给出|需求评审必须|
|B　　　|文本输入框的`normal`，`hover`，`focus`三态|视觉交付时一并给出|需求评审必须|
|B　　　|列表展示不能是绝对完美的状态，需要有多行的情况|视觉交付时一并给出|需求评审必须|
|B　　　|数据展示时如果内容为空时的提示|视觉交付时一并给出|需求评审必须|
|B　　　|`标题`、`内容的字体`、`字体大小`、`行高`、`颜色`|视觉交付时一并给出| 需求评审必须|
|B　　　|各种信息提示的视觉，比如说`提示`、`错误`、`成功`等| 视觉/交互提供模版| 开发侧提供常见出错场景|
|B　　　|转菊花或者进度条视觉| 视觉/交互提供模版| 需求评审必须|
|C　　　|如果更换设计师，请尽量保持以前的细节风格跟进设计|如有变化请告知，并保持整站一致|需求评审必须|
|C　　　|如果更换设计师，请设尽量保持以前的视觉的分辨率跟进设计|如有变化请告知，并保持整站一致|需求评审必须|
|C　　　|涉及到多页面；视觉风格保持|多页面视觉风格保持一致|需求评审必须|
|C　　　|PSD文件图层整理|是否有整理得有顺序，不凌乱？|需求评审必须；由视觉童鞋把关|
|C　　　|PSD文件命名|命名有意义，能大致告诉别人视觉稿内容|需求评审必须；由产品童鞋把关|
|D　　　|多分辨率支持考虑|如果涉及到多分辨率，视觉需要考虑|需求评审必须|
|D　　　|移动端视觉分辨率|手机默认`640x960`分辨率|需求评审必须|
|D　　　|给出的视觉中常用的字体尽量用`微软雅黑`这种字体|视觉稿是否有考虑到？|部分火星文，在客户端上需用Tahoma字体，不然会乱码；开发侧提供会乱码的场景，由视觉/产品把关，替换成Tahoma字体是否可接受|
|D　　　|隐藏图层显现考虑|视觉图层需要保持完整性|需求评审必须；移动图层在交互上移走，导致下方图层展现，需要保证下方图层可用|