---
layout: post
title: jQuery에서 method chaining
date: 2018-07-17 14:00:00 +0900
categories: blog
---


```javascript
$('.example-class').on('mouseenter', () => {
  $('.example-class-one').show();
}).on('mouseleave', () => {
  $('.example-class-one').hide();
});
```



In the example above, we chain a mouse enter event to a mouse leave event. Both of the event handlers target `.example-class` elements. When a user's mouse enters an `.example-class` element's area we show `.example-class-one` elements. When a user's mouse leaves an `.example-class` element's area, we hide `.example-class-one`elements.

This way, browsers do not have to find the same element(s) more than once.



```javascript
// Chaining events used with event and event.currentTarget

$('.btn').on('mouseover', event => {
  $(event.currentTarget).addClass('btn-hover')
}).on('mouseleave', event => {
  $(event.currentTarget).removeClass('btn-hover')
});
```



When chaining, the line of code could become quite long. However, jQuery is not very strict on the syntax; you can format it like you want, including line breaks and indentations. jQuery throws away extra whitespace and executes the lines above as one long line of code.



```javascript
$("#p1").css("color", "red").slideUp(2000).slideDown(2000);
// the code above may be re-written as follow:

$('#p1').css('color', 'red')
	.slideUp(2000)
	.slideDown(2000);
```