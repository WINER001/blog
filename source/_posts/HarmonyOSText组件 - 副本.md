---
title: HarmonyOS_Text组件
date: 2023/12/14 10:12
tags: 
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/41.jpg

---



# HarmonyOS_Text组件



## Text

Text是用来显示字符串的组件，在界面上显示为一块文本区域。Text座位一个基本组件，有很多扩展，常见的有按钮组件Button，文本编辑组件

TextField。



## 支持的XML属性

Text的共有XML属性继承自：Component

Text的自由XML属性见下表：

### 表一，Text的自有XML属性

| 属性名称                       | 中文描述                                                     | 取值说明                                                     | 使用案例                                                     |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| text                           | 显示文本                                                     | string类型。可以直接设置文本字串，也可以引用string资源（推荐使用）。 | ohos:text="熄屏时间"<br />ohos:text ="$string.test_str"      |
| hint                           | 提示文本                                                     | string类型。可以直接设置文本字串，也可以应用string资源（推荐使用）。 | ohos:hint="联系人"<br />ohos:hint="$string:test_str"         |
| text_font                      | 字体                                                         | 可设置值包括：sans-serif，sans-serif-medium,HwChinese-medium,sans-serif-condensed,sans-serif-condensed-medium,monospace | ohos:text_font="HwChinese-medium"                            |
| trunaction_mode                | 长文本截断方式                                               | 1.none：表示文本超长时无截断。<br />2.ellipsis_at_start：表示文本超长时在文本框起始处使用省略号截断<br />3.ellipsis_at_middle：表示文本超长时在文本框结尾处使用省略号截断<br />4.ellipsis_at_end：表示文本超长时在文本框结尾处使用省略号截断。<br />5.auto_scrolling：表示文本超长时滚动显示全部文本。 | 1.ohos:truncation_mode="none"<br />2.ohos:truncation_mode="ellipsis_at_start"<br />3.ohos:truncation_mode="ellipsis_at_middle"<br />4.ohos:truncation_mode="ellipsis_at_end"<br />5.ohos:truncation_mode="auto_scrolling" |
| text_size                      | 文本大小                                                     | float类型。可以是浮点数值，其默认单位为px；也可以是带px/vp/fp单位的浮点数值；也可以引用float资源。 | ohos:element_padding="20"<br />ohos:element_padding="8vp"<br />ohos:element_padding="$float:size_value" |
| bubble_width                   | 文本气泡宽度                                                 | float类型。可以是浮点数值，其默认单位为px；也可以是带px/vp/fp单位的浮点数值；也可以引用float资源。 | ohos:bubble_width="20"<br />ohos:bubble_width="10vp"<br />ohos:bubble_width="$float:size_value" |
| bubble_height                  | 文本气泡高度                                                 | float类型。可以是浮点数值，其默认单位为px；也可以是带px/vp/fp单位的浮点数值；也可以引用float资源。 | ohos:bubble_height="20"<br />ohos:bubble_height="10vp"<br />ohos:bubble_height="$float:size_value" |
| bubble_left_width              | 文本气泡左宽度                                               | float类型。可以是浮点数值，其默认单位为px；也可以是带px/vp/fp的单位的浮点数值；也可以引用float资源。 | ohos:bubble_left_width="20"<br />ohos:bubble_left_width="10vp"<br />ohos:bubble_left_width="$float:size_value" |
| bubble_left_height             | 文本气泡左高度                                               | float类型。可以是浮点数值，其默认单位是px；也可以是带px/vp/fp的的单位的浮点数值；也可以引用float资源。 | ohos:bubble_left_height="20"<br />ohos:bubble_right_width="10vp"<br />ohos:bubble_right_width="$float:size_value" |
| bubble_right_height            | 文本气泡右高度                                               | float类型。额可以是浮点数值，其默认单位是px；也可以是带px/vp/fp的单位的浮点数值；也可以引用color资源。 | ohos:bubble_right_height="20"<br />ohos:bubble_right_height="10vp"<br />ohos:bubble_right_height="$float:size_value" |
| text_color                     | 文本颜色                                                     | color类型。可以直接设置色值，也可以引用color资源。           | ohos:text_color="#A8FFFFFF"<br />ohos:text_color="$color:black" |
| hint_color                     | 提示文本颜色                                                 | color类型。可以直接设置色值，也可以引用color资源。           | ohos:hint_color="#A8FFFFFF"<br />ohos:hint_color="$color:black" |
| selection_color                | 选中文本颜色                                                 | color类型。可以直接设置色值，也可以引用color资源。           | ohos:selection_color="#A8FFFFFF"<br />ohos:selection_color="$color:black" |
| text_alignment                 | 文本对齐方式                                                 | left：表示文本靠左对齐。<br />top：表示文本靠顶部对齐。<br />right：表示文本靠右部对齐。<br />bottom：表示文本靠底部对齐。<br />horizontal_center：表示文本水平居中对齐<br />vertical_center：表示文本垂直居中对齐<br />center：表示文本居中对齐。<br />start：表示文本靠起始段对齐。<br />end：表示文本靠结尾段对齐。 | 可以设置取值项如表中所列，也可以使用“\|”进行多项组合。<br />ohos:text_alignment="top"<br />ohos:text_alignment="topleft" |
| max_text_lines                 | 文本最大行数                                                 | integer类型。可以直接设置整型数值，也可以引用integer资源。   | ohos:max_text_lines="2"<br />ohos:max_text_lines="$integer:two" |
| text_input_type                | 文本输入类型                                                 | pattern_null：表示未指定文本输入类型，默认文本输入类型为内容模式。<br />pattern_text：表示文本输入类型为普通文本模式。<br />pattern_number：表示文本输入类型为数字。<br />pattern_password：表示文本输入类型为密码。 | ohos:text_input_type="pattern_null"<br />ohos:text_input_type="pattern_text"<br />ohos:text_input_type="pattern_number"<br />ohos:text_input_type="pattern_password" |
| input_enter_key_type           | 输入键类型                                                   | enter_key_type_unspecified：表示未指定输入键类型，采用默认类型。<br />enter_key_type_search：表示采用执行“搜索”动作的输入键类型。<br />enter_key_type_go：表示采用执行"go"动作的输入键类型。<br />enter_key_type_send：表示采用执行”发送“动作的输入键类型。 | ohos:input_enter_key_type="enter_key_type_unspecified"<br />ohos:input_enter_key_type="enter_key_type_search<br />ohos:input_enter_key_type="enter_key_type_go"<br />ohos:input_enter_key_type="enter_key_type_send" |
| auto_scrolling_duration        | 自动滚动时长                                                 | integer类型：可以直接设置整型数值，也可以引用integer资源。<br />表示时间的值不可小于0，单位为ms。 | ohos:auto_scrolling_duration="1000"<br />ohos:auto_scrolling_duration="$integer:during" |
| multiple_lines                 | 多行模式设置                                                 | boolean类型。可以直接设置true/false,也可以引用boolean资源    | ohos:multiple_lines="true"<br />ohos:multiple_lines="$boolean:true" |
| auto_font_size                 | 是否支持文本自动调整文本字体大小                             | boolean类型。可以直接设置true/false,也可以引用boolean资源。  | ohos:auto_font_size="true"<br />ohos:auto_font_size="$boolean:true" |
| scrollable                     | 文本是否可滚动                                               | boolean类型。可以直接设置true/false，也可以引用boolean资源。 | ohos:scrollable="true"<br />ohos:scrollable="$boolean:true"  |
| text_cursor_visible            | 文本光标是否可见。只有在可编辑的组件上可配置，否则该值始终为false。 | boolean类型。可以直接设置true/false，也可以引用boolean资源。 | ohos:text_cursor_visible="true"<br />ohos:text_cursor_visible="$boolean:true" |
| italic                         | 文本是否斜体字体                                             | boolean类型。可以直接设置true/false，也可以引用boolean资源。 | ohos:italic="true"<br />ohos:italic="$boolean:true"          |
| padding_for_text               | 设置文本顶部与底部是否默认留白。默认值为true，true表示保留默认留白，false表示顶部与底部不留白 | boolean类型。可以直接设置true/false,也可以引用boolean资源。  | ohos:padding_for_text="true"<br />ohos:padding_for_text="$boolean:true" |
| additional_line_spacing        | 需增加的行间距                                               | float类型。可以直接设置浮点数值，也可以引用float资源。       | ohos:additional_line_spacing="2"<br />ohos:additional_line_spacing="$float:line_spacing_add" |
| line_height_num                | 行间距倍数                                                   | float类型。可以直接设置浮点数值，也可以引用float资源。       | ohos:line_height_num="1.5"<br />ohos:line_height_num="$float:line_spacing_multi" |
| element_left                   | 文本左侧图标                                                 | Element类型。可直接配置色值，也可以引用color资源或引用media/graphic下的图片资源。<br />element_left属性冲突说明：<br />1.element_left与element_start，element_end属性有冲突，不建议一起使用。在”水平布局方向为从左到右“时，element_left会与element_start属性冲突；在”水平布局方向为从右到左时，element_left会与element_end属性冲突。<br />2.同时配置时，element_start，element_end优先级高于element_left属性。 | ohos:element_left="#FFFFFFFF"<br />ohos:element_left="$color:black"<br />ohos:element_left="$media:media_src"<br />ohos:element_left="$graphic:graphic_src" |
| element_top                    | 文本上方图标                                                 | Element类型。可直接配置色值，也可引用color资源或引用media/graphic下的图片资源。 | ohos:element_top="#FFFFFFFF"<br />ohos:element_top="$color:black"<br />ohos:element_top="$media:media_src"<br />ohos:element_top="$graphic:graphic_src" |
| element_right                  | 文本右侧图标                                                 | Element类型。可直接配置色值，也可引用color资源或引用media/graphic下的图片资源。<br />element_right属性冲突说明：<br />1.element_right与element_start，element_end属性有冲突，不建议一起使用。在“水平布局方向为从左到右”时，element_right会与element_end属性冲突；在“水平布局方向为从右到左”时，element_right会与element_start属性冲突。<br />2.同时配置时，element_start，element_end优先级高于element_right属性。 | ohos:element_right="#FFFFFFFF"<br />ohos:element_right="$color:black"<br />ohos:element_right="$media:media_src"<br />ohos:element_right="$graphic:graphic_src" |
| element_bottom                 | 文本下方图标                                                 | Element类型。可直接配置色值，也可引用color资源或引用media/graphic下的图片资源 | ohos:element_bottom="#FFFFFFFF"<br />ohos:element_bottom="$color:black"<br />ohos:element_bottom="$media:media_src"<br />ohos:element_bottom="$graphic:graphic_src" |
| element_start                  | 文本开始方向图标                                             | Element类型。可直接配置色值，也可引用color资源或引用media、graphic下的图片资源。<br />element_start属性冲突说明<br />1.element_start与element_left，element_right属性有冲突，不建议一起使用。在“水平布局方向为从左到右”时，element_start会与element_left属性冲突；在“水平布局方向为从右到左”时，element_start会与element_right属性冲突。<br />2.同时配置时，element_start优先级高于element_left，element_right属性。 | ohos:element_start="#FFFFFFFF"<br />ohos:element_start="$color:black"<br />ohos:element_start="$media:media_src"<br />ohos:element_start="$graphic:graphic_src" |
| element_end                    | 文本结束方向图标                                             | Element类型。可直接配置色值，也可引用color资源或引用media/graphic下的图片资源。<br />element_end属性冲突说明<br />1.element_end与element_left，element_right属性有冲突，不建议一起使用。在“水平布局方向为从左到右”时，element_end会与element_left属性冲突。<br />2.同时配置时，element_end优先级高于element_left，element_right属性 | ohos:element_end="#FFFFFFFF"<br />ohos:element_end="$color:block"<br />ohos:element_end="$media:media_src"<br />ohos:element_end="$graphic:graphic_src" |
| element_cursor_bubble          | 文本的光标气泡图形<br />只有在可编辑的组件上可配置           | Element类型。可直接配置色值，也可引用color资源或引用media/graphic下的图片资源 | ohos:element_cursor_bubble="#FFFFFFFF"<br />ohos:element_cursor_bubble="$color:black"<br />ohos:element_cursor_bubble="$media:media_src"<br />ohos:element_cursor_bubble="$graphic:graphic_src" |
| element_selection_left_bubble  | 选中文本的左侧气泡图形                                       | Element类型。可直接配置色值，也可引用color资源或引用media/graphic下的图片资源。 | ohos:element_selection_left_bubble="#FFFFFFFF"<br />ohos:element_selection_left_bubble="$color:black"<br />ohos:element_selection_left_bubble="$media:media_src"<br />ohos:element_selection_left_bubble="$graphic:graphic_src" |
| element_selection_right_bubble | 选中文本的右侧气泡图形                                       | Element类型。可直接配置色值，也可引用color资源或引用media/graphic下的图片资源。 | ohos:element_selection_right_bubble="$FFFFFFFF"<br />ohos:element_selection_right_bubble="$color:black"<br />ohos:element_selection_right_bubble="$media:media_src"<br />ohos:element_selection_right_bubble="$graphic:graphic_src" |



官方文档：https://developer.harmonyos.com/cn/docs/documentation/doc-guides/ui-java-component-text-0000001050729534?spm=a2c6h.12873639.article-detail.7.40317ec3RXS4JE
