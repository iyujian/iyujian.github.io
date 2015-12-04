---
layout: post
title:  "设计模式之——策略模式（Head first设计模式学习笔记）"
date:   2015-12-04 14:20:45 +0800
categories: java
tags: design patterns
---
1、 策略模式的定义：

策略模式定义了算法簇，分别分装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

2、 设计原则：

1) 找出应用中可能需要变化之处，把它们独立出来，不要和那些不需要变化的代码混在一起；

2) 针对接口编程，而不是针对实现编程；

3) 多用组合，少用继承；

<!-- more -->

3、 实现：

假设我们现在有两个角色，骑士和巨魔，骑士使用弓箭做武器，巨魔使用斧头当武器，那么我们如何进行设计呢？

1) 首先我们应该有个武器行为的接口：WeaponBehavior，它有一个use()方法来使用武器。

{% highlight java linenos %}
package designpatterns.strategypattern.behavior;

/**
 * 武器行为接口
 */
public interface WeaponBehavior {

    void use();

}
{% endhighlight %}

2) 这个接口有两个实现类，分别是弓箭(BowAndArrowBehavior)和斧头(AxeBehavior)。
{% highlight java linenos %}
package designpatterns.strategypattern.behavior;

/**
 * 弓箭行为
 */
public class BowAndArrowBehavior implements WeaponBehavior {
    @Override
    public void use() {
        System.out.println("我正在使用弓箭射击！");
    }
}

{% endhighlight %}

{% highlight java linenos %}
package designpatterns.strategypattern.behavior;

/**
 * 斧头行为
 */
public class AxeBehavior implements WeaponBehavior {
    @Override
    public void use() {
        System.out.println("我正在使用斧头劈砍！");
    }
}

{% endhighlight %}


3) 然后我们应该有一个角色的抽象父类（超类，super class）。这个类有一个成员变量 weaponBehavior（武器行为），还有一个fight()方法，通过调用这个方法来调用武器的行为，还有个setWeaponBehavior()方法，可以来动态的设定武器行为。

{% highlight java linenos %}
package designpatterns.strategypattern;

import designpatterns.strategypattern.behavior.WeaponBehavior;

/**
 * 角色类，超类
 */
public abstract class Character {

    protected WeaponBehavior weaponBehavior;

    public void fight() {
        weaponBehavior.use();
    }

    public void setWeaponBehavior(WeaponBehavior weaponBehavior) {
        this.weaponBehavior = weaponBehavior;
    }
}
{% endhighlight %}

4) 接下来，骑士和巨魔要继承上面的父类。他们都有个构造方法，来初始化他们的武器。

{% highlight java linenos %}
package designpatterns.strategypattern;

import designpatterns.strategypattern.behavior.BowAndArrowBehavior;

/**
 * 骑士
 */
public class Knight extends Character {

    public Knight() {
        this.weaponBehavior = new BowAndArrowBehavior();
    }

}
{% endhighlight %}

{% highlight java linenos %}
package designpatterns.strategypattern;

import designpatterns.strategypattern.behavior.AxeBehavior;

/**
 * 巨魔
 */
public class Troll extends Character {

    public Troll() {
        this.weaponBehavior = new AxeBehavior();
    }

}
{% endhighlight %}

5) 做个测试

{% highlight java linenos %}
Character character = new Knight();
character.fight();

character = new Troll();
character.fight();
{% endhighlight %}

输出结果符合预期：

```
我正在使用弓箭射击！
我正在使用斧头劈砍！
```

现在想让骑士也用斧头来当武器那么只需要调用 setWeaponBehavior() 方法来动态的设定武器行为。

{% highlight java linenos %}
Character character = new Knight();
character.setWeaponBehavior(new AxeBehavior());
character.fight();
{% endhighlight %}

输出

```
我正在使用斧头劈砍！
```


6) 现在又有个人来了，它是国王，能使用宝剑，那么，我们只需要让国王 King 继承 Character，让宝剑 Sword 实现 WeaponBehavior 就可以了。

{% highlight java linenos %}
package designpatterns.strategypattern;

import designpatterns.strategypattern.behavior.SwordBehavior;

/**
 * 国王
 */
public class King extends Character {

    public King() {
        this.weaponBehavior = new SwordBehavior();
    }

}
{% endhighlight %}

{% highlight java linenos %}
package designpatterns.strategypattern.behavior;

/**
 * 宝剑行为
 */
public class SwordBehavior implements WeaponBehavior {
    @Override
    public void use() {
        System.out.println("我正在使用宝剑挥舞！");
    }
}
{% endhighlight %}

7) 再来测试下：

{% highlight java linenos %}
Character character = new King();
character.fight();
{% endhighlight %}

```
我正在使用宝剑挥舞！
```