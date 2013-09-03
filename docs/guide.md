
## Writing Middleware

  Koa middleware are simple functions which return a `GeneratorFunction`, and accept another. When
  the middleware is run by an "upstream" middleware, it must manually `yield` to the "downstream" middleware.

  For example if you wanted to track how long it takes for a request to propagate through Koa by adding an
  `X-Response-Time` header field the middleware would look like the following:

```js
function responseTime(next){
  return function *(){
    var start = new Date;
    yield next;
    var ms = new Date - start;
    this.set('X-Response-Time', ms + 'ms');
  }
}

app.use(responseTime);
```

  If you're a front-end developer you can think any code before `yield next;` as the "capture" phase,
  while any code after is the "bubble" phase. Here's another way to write the same thing, inline:

```js
app.use(function(next){
  return function *(){
    var start = new Date;
    yield next;
    var ms = new Date - start;
    this.set('X-Response-Time', ms + 'ms');
  }
});
```

  Next we'll look at the best practices for creating Koa middleware.

## Middleware Best Practices

  When creating public middleware it's useful to conform to the convention of
  wrapping the middleware in a function that accepts options, allowing users to
  extend functionality. Even if your middleware accepts _no_ options, this is still
  a good idea to keep things uniform.

  Here our contrived `logger` middleware accepts a `format` string for customization,
  and returns the middleware itself:

```js
function logger(format){
  format = format || ':method ":url"';

  return function(next){
    return function *(){
      var str = format
        .replace(':method', this.method)
        .replace(':url', this.url);

      console.log(str);
      
      yield next;
    }
  }
}

app.use(logger());
app.use(logger(':method :url'));
```


