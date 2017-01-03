---
layout: post
title:  "What is Promise"
date:   2017-01-02 20:46:58 -0600
comments: true
archive: true
categories: Javascript
---

> The core idea behind promises is that a promise represents the result of an asynchronous operation. A promise is in one of three different states:
>
 * pending - The initial state of a promise.
 * fulfilled - The state of a promise representing a successful operation.
 * rejected - The state of a promise representing a failed operation.
>
> Once a promise is fulfilled or rejected, it is immutable \(i.e. it can never change again\).

Promise는 callback 을 이용하여 코드를 작성하는 경우 점점 depth가 늘어나면서 복잡해져가는 코드를 가독성을 높여주고 callback 사용으로 인해 call stack이 의도치 않은 방향으로 흐르는 것을 방지하고 개발자에게 error handling 을 할 수 있는 권한을 돌려 준 모듈이라고 인터넷에 정의되어 있는 것 같다.

[We have a problem with Promises][We-have-a-problem-with-Promises] article 을 읽어보면 어떻게 Promise를 써야 할 것인지 조금은 감이 잡히는 것 같다.

javascript 코드를 작성할 때 너무 많은 depth를 가지는 형태로 코드를 작성하는 피해야 한다.

Promise는 synchronous code나 value를 감싸 asynchronous 형태로 만들어준다. 방식은 아래와 같이,

{% highlight javascript %}
new Promise( function( resolve, reject ){
  resolve( someSynchronousValue );
} );
{% endhighlight %}

{% highlight javascript %}
Promise.resolve( someSynchronousValue ).then( );
{% endhighlight %}

또는, 아래와 같이 function을 작성할 때에 Promise를 리턴하도록 작성하면 함수 호출 후 .then\(\) 또는 .catch\(\) 를 사용하여 처리가 가능하기에 이런 방법을 추천한다.

{% highlight javascript %}
function somePromiseAPI(){
  return Promise.resolve().then( function(){
    doSomethingThatMayThrow();
    return 'foo';
  } ).then(/*.....*/);
}
{% endhighlight %}

then() 에 정의된 function은 반드시 return을 해 주어야 다음 .then()에서 값을 전달 받을 수 있다는 것을 명심하자.

[We-have-a-problem-with-Promises]: https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html?utm_source=javascriptweekly&utm_medium=email