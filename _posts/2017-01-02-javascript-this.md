---
layout: single
title:  "this in Javascript"
comments: true
archive: true
date:   2017-01-02 23:53:58 -0600
categories: Javascript
tags: javascript this
---

node에서는 global object를 브라우저에서는 window object를 의미한다. 하지만 strict mode 에서는 undefined

>In the global execution context (outside of any function), this refers to the global object, whether in strict mode or not.

{% highlight javascript %}
function f1(){
  return this;
}
// In a browser:
f1() === window; // the window is the global object in browsers

// In Node:
f1() === global
{% endhighlight %}

strict mode에서는 function을 실행할 때 넣어주는 값이 this의 값으로 유지되기 때문에 아래 코드의 경우 this는 undefined 가 된다.

{% highlight javascript %}
function f2(){
  "use strict"; // see strict mode
  return this;
}

f2() === undefined;
{% endhighlight %}

strict mode에서 function call을 call() 또는 apply() 를 이용하여 this object를 전송해 주면 this 사용이 가능하다.

{% highlight javascript %}
(function(){
  'use strict';

  function f3(){
    return this;
  }
  f3.call(this) === window; // global object

  function f4(){
    this.herp = "derp";
  }
  function Thing(){
    this.thisIsEasyToUnderstand = "just kidding";
    f4.call(this);
  }
  var thing = new Thing();
  // thing = { thisIsEasyToUnderstand : "just kidding", herp: "derp" };

})();
{% endhighlight %}
