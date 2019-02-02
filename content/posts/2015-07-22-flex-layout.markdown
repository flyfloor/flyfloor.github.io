---
date: "2015-07-22"
layout: post
tags: ['collection', 'tech', 'Css', 'flex-layout']
title: Getting Dicey With Flexbox
---

> 此文来自*[Getting Dicey With Flexbox](//davidwalsh.name/flexbox-dice)*, 原文作者: Landon Schropp 

<!-- more -->

What if you could build complex CSS layouts in minutes? Flexbox is a new CSS layout spec that makes it easy to construct dynamic layouts. With flexbox, vertical centering, same-height columns, reordering, and direction agnosticism are a piece of cake.  

There's a popular myth floating around that flexbox isn't ready for prime time. Wrong! **93% of people are now running a browser that supports flexbox. **That's better than the support for the HTML5 ```<video>``` element.  

In this article, I’ll walk you through the basics of flexbox by showing you how to build the faces of a die. For now, we'll skip some of the trickier parts of flexbox such as vendor prefixes, old versions of the syntax and browser quirks. I’ll cover those in upcoming lessons.  

This is the first lesson in a seven-part series on flexbox. If you like this article, be sure to subscribe to the entire course over at [Free Flexbox Starter Course](https://unravelingflexbox.com/subscribe?utm_source=David%20Walsh%20Blog&utm_medium=Blog&utm_campaign=Unraveling%20Flexbox).  

### The First Face  
A die consists of six faces (sides). Each face has a number of pips (dots) which determine the value of the side. The first side consists of a single pip in the center of the face.  

Let's start with the HTML.  

```html
<div class="first-face">
  <span class="pip"></span>
</div>
```

To make life a little easier, I’ve added the basic styles for the faces and the pips. Here's what this face looks like:  

![](/images/flex/1.png)  

The first step is to tell the browser to make the face a flexbox container.  

```css
.first-face {
  display: flex;
}
```

![](/images/flex/2.png)  

That doesn't look any different, but there's a lot going on under the hood.  

![](/images/flex/3.png)  

The **first-face** container now has a horizontal main axis. The main axis of a flexbox container can be either horizontal or vertical, but the default is horizontal. If we added another pip to the face, it would show up to the right of the first one. The container also has a vertical cross axis. The cross axis is always perpendicular to the main axis.  

The **justify-content** property defines the alignment along the main axis. Since we want to center the pip along the main axis, we'll use the **center** value.  

```css
.first-face {
  display: flex;
  justify-content: center;
}
```  

![](/images/flex/4.png)    

Hey, cool! Since the main axis is horizontal, the pip is now centered in the parent element.  

The **align-items** property dictates how the items are laid out along the cross axis. Because we want the pip to center along this axis, we'll use the center value here too.  

```css
.first-face {
  display: flex;
  justify-content: center;
  align-items: center;
}
```  

![](/images/flex/5.png)  

And just like that, the pip is centered!  

### Getting Trickier  
On the second face of a die, the first pip is in the top left corner and the second is in the bottom right. This is also pretty easy to do with flexbox!    
Again, let's start with the markup and the basic CSS.  

```html
<div class="second-face">
  <span class="pip"></span>
  <span class="pip"></span>
</div>
```  

```css
.second-face {
  display: flex;
}
```  

![](/images/flex/6.png)  

Now we have two pips right next to each other. This time around, we want the pips on opposite sides of the die. There's a value for **justify-content** that will let us do just that: **space-between**.  

The **space-between** property evenly fills the space between flex items. Since there are only two pips, this pushes them away from each other.  

```css
.second-face {
  display: flex;
  justify-content: space-between;
}
```  

![](/images/flex/7.png)  

Here's where we run into a problem. Unlike before, we can't set align-items because it will affect both pips. Luckily, flexbox includes align-self. This property lets us align individual items in a flex container along the cross axis however we'd like! The value we want for this property is **flex-end**.  

```css
.second-face {
  display: flex;
  justify-content: space-between;
}  
.second-face .pip:nth-of-type(2) {
  align-self: flex-end;
}
```

![](/images/flex/8.png)  

Looks good!  

### Horizontal and Vertical Nesting  
Let's skip the third face and tackle the fourth. This one is a little trickier than the others because we need to support two columns, each with two pips.

There are two things about flexbox that will save us here: flex containers can have vertical or horizontal content, and flex containers can be nested.

Unlike before, our markup will now include columns.

```html
<div class="fourth-face">
  <div class="column">
    <span class="pip"></span>
    <span class="pip"></span>
  </div>
  <div class="column">
    <span class="pip"></span>
    <span class="pip"></span>
  </div>
</div>
```  

![](/images/flex/9.png)  

Since we want the two columns to be on opposite sides, let's go ahead and use justify-content: space-between like we did before.

```css
.fourth-face {
  display: flex;
  justify-content: space-between;
}
```  

![](/images/flex/10.png)  

Next, we need to make the columns flex containers. It might seem like they already are, but remember that we haven't set **display: flex** yet. We can use the flex-direction property to set the direction of the main axis to column.  

```css
.fourth-face {
  display: flex;
  justify-content: space-between;
}  
.fourth-face .column {
  display: flex;
  flex-direction: column;
}
```

![](/images/flex/11.png)  

t doesn't look any different, but the columns are now flex containers. Notice how we stuck a flex container directly inside another flex container? That's okay! Flexbox doesn't care if the containers are nested.

The final step is to space the the pips inside the columns apart from each other. Since the main axis is vertical, we can use **justify-content** again.  

```css
.fourth-face {
  display: flex;
  justify-content: space-between;
}  
.fourth-face .column {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
```

![](/images/flex/12.png)  

*Note: This face could have been built without columns by using wrapping. I'll cover wrapping in more detail in a future lesson.*  

### Wrapping up  
Woohoo! Three faces down and three to go. At this point, you should have everything you need to build the other three. Your homework is to write the CSS for the missing faces. When you're done, you can look at the CodePen below for the answers. Let me know how it went in the comments!  

See the Pen [Flexbox Dice](//codepen.io/LandonSchropp/pen/KpzzGo/) by Landon Schropp [(@LandonSchropp)](//codepen.io/LandonSchropp) on CodePen).  

My next lesson will cover how to write flexbox without vendor prefixes or old syntax. To make sure you get it, head over to [Free Flexbox Starter Course](https://unravelingflexbox.com/subscribe?utm_source=David%20Walsh%20Blog&utm_medium=Blog&utm_campaign=Unraveling%20Flexbox) and sign up.