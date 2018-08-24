title: linear-grandient 制作斜切角
date: 2018/8/13
categories:
- css
----------

#### linear-grandient 制作斜切角

* 效果图
![1.jpg](./1.jpg)

* 代码

    ```css
    padding: 14px;
    width: 200px;
    height: 100px;
    background: #DDD;
    border-radius: 4px;
    background: linear-gradient(-45deg, transparent 25px, #DDD 0)
    ```

* 说明

    * linear-grandient 各参数分别是 angle，[color length]
    * angle 为角度
    * [color length] 为颜色和长度结对出现，多组以逗号分隔

* angle 角度说明

    ![2.jpg](./2.jpg)

    * C 为容器中心点，A则是通过C的垂直线与渐变线顺时针旋转产生的脚，渐近线左下点为起始点，右上点为结束点

    * 如上例中，角度为 -45deg时，渐近线逆时针旋转45度，渐进方向则为右下到左上

* 进阶

    * 使用 linear-grndient 还可以实现以下效果

    ![3.jpg](./3.jpg)

    * 代码

    ```css
    border-radius: 6px;
    background: #CB34DC;
    width: 120px;
    height: 40px;
    background: linear-gradient(-45deg, transparent 15px, #CB34DC 0) bottom right, linear-gradient(-135deg, transparent 15px, #CB34DC 0) top right;
    background-size: 100% 50%;
    background-repeat: no-repeat;
    ```

