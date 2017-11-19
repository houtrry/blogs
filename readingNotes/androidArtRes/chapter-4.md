# 第四章 View的工作流程

## 初识ViewRoot和DecorView 
DecorView作为顶级View，一般情况下，它的内部会包含一个竖直方向的LinearLayout，在这个LinearLayout里面有上下两个部分（具体情况和Android版本及主题有关），上面是标题，下面的是内容栏。在Activity中我们通过setContentView所设置的布局文件其实就是被加载到内容栏之中的，而内容栏的id是content，可以通过ViewGroup content = (ViewGroup)findViewById(R.id.content)来得到content，可以通过content.getChildAt(0)来获取我们设置的内容。

## MeasureSpec
MeasureSpec代表一个32位的int值，低30位代表SpecSize， 高2位代表SpecMode。  
SpecMode有三类：  
* UNSPECIFIED： 父容器不对View有任何限制， 这种情况一般用于系统内部。
* EXACTLY： 对应于LayoutParams中的match_parent和具体的数值两种模式。
* AT_MOST:对应于LayoutParams中的warp_content.



