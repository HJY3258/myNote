- [问题与解决方案](#1)
- [typescript](#2)
- [想法](#3)
- [项目相关](#4)
- - [伏魔](#5)
- - [fgui/Laya](#6)

<h1 id = "1">问题与解决方案</h1>

+ 像是弹窗这种需要高度自定义的东西在调用显示的时候最好不要将参数放在参数列表中，而是放在键值对对象中，这样虽然损失一些调用时的可读性，但至少扩展的时候不至于因为要添加更长参数列表而感到烦恼，而且新添加的参数也更隐形和自由，不会因为不小心搞乱参数列表使其他地方出现问题

<h1 id = "2">Typescript</h1>

```ts
{
    //普通键值对键和值定义类型的语法
    shape_type: { [key: number]: number };
}
```


<h1 id = "3">想法</h1>

+ 对类似的界面可以做个父类，这样既分隔又统一，互相改东西可以互不影响，需要统一修改的东西又可以在父类里做修改

<h1 id = "4">项目相关</h1>

<h2 id = "5"> 伏魔</h2>

+ 战力增加数值滚动实现：根据目标战力显示，每帧对其中每一位数字随机生成一个数字然后显示，进行一定时间或次数的滚动后显示目标str。

<h2 id = "6"> fgui/Laya</h2>

+ 文字的显示流程

```ts
{
    //fgui中
    //普通文本和富文本都有一个共同父类
    GTextField//它果然还是作为抽象定义，负责的具体逻辑不多
    GBasicTextField //它是普通文本，子类之一
    public set text(value: string)//赋值从这里进去，这里主要是做文本类型判断以及做ubb富文本解析
    this._textField.typeset() //然后进入到这个方法中而这里就是Laya.Text的地盘了，往下看
    
}
```
```ts
{
    //laya中
    //Text类
    typeset(): void {
         //进入到这里会做文本排版，设置字体
        ...
        //设置字体
        ILaya.Browser.context.font = this._getContextFont();
        
        ...
        if (this._isPassWordMode())//如果是password显示状态应该使用密码符号计算
        {
            this._parseLines(this._getPassWordTxt(this._text));
        } else
            //计算文本换行位置，包括自动换行富文本换行
            this._parseLines(this._text);
        
        //计算每行的宽高
        this._evalTextSize();
        //启用Viewport
        if (this._checkEnabledViewportOrNot()) this._clipPoint || (this._clipPoint = new Point(0, 0));
        //否则禁用Viewport
        else this._clipPoint = null;
        //渲染文本
        this._renderText();
    }

    /**
     * @private
     * 渲染文字。
     * @param	begin 开始渲染的行索引。
     * @param	visibleLineCount 渲染的行数。
     */
    protected _renderText(): void {
        ...
        var visibleLineCount = this._lines.length;

        var beginLine = this.scrollY / (this._charSize.height + this.leading) | 0;

        //清空绘制指令（回想图形学课设应该就能理解）
        var graphics = this.graphics;
        graphics.clear(true);

        ...

        //处理垂直对齐
        ...

        //处理水平对齐
        ...

        //计算缩放和开始pos
        ...

        //渲染
        if (this._clipPoint) {
            //绘制画布宽高等信息
            ...
        }

        //从开始行到最后一行
        for (let i = beginLine; i < end; i++) {
            let word = lines[i];
            let wordText: WordText; //应该是一行文字的对象，会对这行文字做成图集缓存

            ...

            //判断是否是位图字体
            if (bfont) {
                ...
                //是的话拿着位图字体对象直接绘制，类似图片绘制了这里就，一个位图字体只有一张图集，不需要想普通文本一样动态生成图集，所以不需要额外处理
                bfont._drawText(word, this, x, y, this.align, tWidth, this._color);
            } else {
                //容错处理
                ...
                wordText.setText(word);
                //是否拆分渲染
                wordText.splitRender = this._singleCharRender;
                //绘制指令 当然有描边和无描边用的都是FillTextCmd这个绘制命令，只是参数不一样
                graphics.fillText(wordText, x, y, ctxFont, this._color, textAlgin);
            }
        }
        //缩放与保存graphics
        ...

    }

    //接下来就是创建cmd命令流，命令流run的时候其实就是context里的textrender调_fast_filltext来绘制文本

    //之后就不是太理解了，大概就是将要显示的文本绘制在一张贴图里，然后用_inner_drawTexture的方法来绘制图片，最后其实还是绘制图片。
    //然后还有就是一些优化操作比如整句渲染的话就做成一张贴图，这是就需要去贴图集里查是否已经存在这张帖图，如果已经存在就用已有的，没有就生成，
    //如果是分字渲染就是做个贴图数组，每个字一个贴图，这句话有相同的字就归并，不重复生成之类的。
    
    //ui的显示就是创建一个平面的网格，然后把贴图放上去，之后更深入的就不是现在的我能看懂的了
}
```