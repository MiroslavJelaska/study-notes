# Event loop
##▶ [Philip Roberts: What the heck is the event loop anyway?](http://2014.jsconf.eu/speakers/philip-roberts-what-the-heck-is-the-event-loop-anyway.html) ★★★

* Js what are you? -> A single-threaded non-blocking asynchronous concurrent language
* terms: JS Runtime (heap + stack), WebAPIs, render queue, callback queue, event loop
* render queue has higher priority than callback queue
* Following timeouts won't run after 1000ms. 1000ms is least time required for execution
<pre><code>setTimeout(function(){ console.log('hi') }, 1000);
setTimeout(function(){ console.log('hi') }, 1000);
setTimeout(function(){ console.log('hi') }, 1000);
</code></pre>
* Synchronous -> asynchrounous
<pre><code>//Synchronous
[1, 2, 3, 4].forEach(function(i){ console.log(i); });
//Asynchrounous
function asyncForEach(array, cb){
  array.forEach(function(){
    setTimeout(cb, 0);  
  });
}
asyncForEach([1, 2, 3, 4], function(i) { console.log(i); });</code></pre>

##🔗 [MDN:Concurrency model and Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)
* Run-to-completion
* A web worker or a cross-origin iframe has its own stack, heap, and message queue