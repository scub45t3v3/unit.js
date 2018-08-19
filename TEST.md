# TOC
   - [Faking time](#faking-time)
   - [supertest library to as httpAgent](#supertest-library-to-as-httpagent)
   - [Unit.js](#unitjs)
   - [Control flow (async)](#control-flow-async)
   - [Asserter array()](#asserter-array)
     - [array() behavior](#asserter-array-array-behavior)
     - [Assertions of array()](#asserter-array-assertions-of-array)
   - [Asserter bool()](#asserter-bool)
     - [bool() behavior](#asserter-bool-bool-behavior)
     - [Assertions of bool()](#asserter-bool-assertions-of-bool)
   - [Asserter date()](#asserter-date)
     - [date() behavior](#asserter-date-date-behavior)
     - [Assertions of date()](#asserter-date-assertions-of-date)
   - [Asserter error()](#asserter-error)
     - [error() behavior](#asserter-error-error-behavior)
     - [Assertions of error()](#asserter-error-assertions-of-error)
   - [Asserter exception()](#asserter-exception)
     - [exception() behavior](#asserter-exception-exception-behavior)
     - [Assertions of exception()](#asserter-exception-assertions-of-exception)
   - [Asserter function()](#asserter-function)
     - [function() behavior](#asserter-function-function-behavior)
     - [Assertions of function()](#asserter-function-assertions-of-function)
   - [Asserter number()](#asserter-number)
     - [number() behavior](#asserter-number-number-behavior)
     - [Assertions of number()](#asserter-number-assertions-of-number)
   - [Asserter object()](#asserter-object)
     - [object() behavior](#asserter-object-object-behavior)
     - [Assertions of object()](#asserter-object-assertions-of-object)
   - [Asserter regexp()](#asserter-regexp)
     - [regexp() behavior](#asserter-regexp-regexp-behavior)
     - [Assertions of regexp()](#asserter-regexp-assertions-of-regexp)
   - [Asserter string()](#asserter-string)
     - [string() behavior](#asserter-string-string-behavior)
     - [Assertions of string()](#asserter-string-assertions-of-string)
   - [Asserter undefined()](#asserter-undefined)
     - [undefined() behavior](#asserter-undefined-undefined-behavior)
   - [Asserter value()](#asserter-value)
     - [value() behavior](#asserter-value-value-behavior)
     - [Assertions of value()](#asserter-value-assertions-of-value)
   - [Unit.js provides several API unified and several assertion styles](#unitjs-provides-several-api-unified-and-several-assertion-styles)
   - [Control flow](#control-flow)
     - [Performs the tests without entangled with the flow of other series of tests](#control-flow-performs-the-tests-without-entangled-with-the-flow-of-other-series-of-tests)
   - [Passes IOC container](#passes-ioc-container)
   - [Unit.js provides sinon.js](#unitjs-provides-sinonjs)
     - [Spies](#unitjs-provides-sinonjs-spies)
     - [Stubs](#unitjs-provides-sinonjs-stubs)
     - [Testing Ajax](#unitjs-provides-sinonjs-testing-ajax)
     - [Mocks](#unitjs-provides-sinonjs-mocks)
     - [Matchers](#unitjs-provides-sinonjs-matchers)
     - [Sandbox](#unitjs-provides-sinonjs-sandbox)
<a name=""></a>
 
<a name="faking-time"></a>
# Faking time
calls callback after 100ms.

```js
var spy = test.spy();
    setTimeout(spy, 100);
    clock.tick(99);
    test.assert(spy.notCalled);
    clock.tick(1);
    test.assert(spy.calledOnce);
    // Also:
    test.assert.strictEqual(new Date().getTime(), 100);
```

<a name="supertest-library-to-as-httpagent"></a>
# supertest library to as httpAgent
Good request.

```js
var testServer;
    test.given('Create a server asserter', function(){
      testServer = function(name){
        test.bool(indicator.get(name + '_page')).isFalse();
        test.httpAgent(server)
          .get('/' + (name == 'home' ? '' : name))
          .expect(200, name + ' page')
          .expect('x-powered-by', 'unit.js')
          .expect('Content-Type', /text/)
          .end(function(err, res){
            test
              .bool(indicator.get(name + '_page'))
                .isTrue()
              .value(err)
                .isFalsy()
              .string(res.text)
                .isIdenticalTo(name + ' page')
            ;
          })
        ;
      };
    })
    .then('Test the server: "/"', function(){
      testServer('home');
    })
    .then('Test the server: "/some"', function(){
      testServer('some');
    })
    .then('Exit test case', function(){
      done();
    })
    ;
```

Bad request.

```js
test.httpAgent(server)
      .get('/bad')
      .expect(200)
      .end(function(err, res){
        test.object(err).isInstanceOf(Error);
        done();
      })
    ;
```

Bad content.

```js
test.httpAgent(server)
      .get('/')
      .expect(200, 'bad content')
      .end(function(err, res){
        test.object(err).isInstanceOf(Error);
        done();
      });
```

<a name="unitjs"></a>
# Unit.js
inherits from `Noder`.

```js
test
      .object(test)
        .isInstanceOf(Noder)
        .isInstanceOf(api.UnitJS)
        .isIdenticalTo(api)
      .function(test.constructor)
        .hasName('UnitJS')
    ;
```

<a name="control-flow-async"></a>
# Control flow (async)
Based on bluebird.

```js
test
  .function(test.promise)
    .hasName('Promise')
    .isIdenticalTo(Promise)
    .isIdenticalTo(require('bluebird'))
;
```

supports: given, when, then.

```js
var fs = test.promisifyAll(require('fs'));
    test.promise
      .given(function() {
        return fs.readFileAsync(__filename);
      })
      .when(function(raw) {
        test
          .string(raw.toString())
            .isNotEmpty()
            .contains('I m here !!!')
        ;
        return raw.toString();
      })
      .then(function(contents) {
        test
          .string(contents)
            .isNotEmpty()
            .contains('I m here also !!!')
        ;
      })
      .catch(function(err){
        test.fail('test.promise, ' + err.message);
      })
      .finally(done)
      .done()
    ;
```

test.promisifyAll().

```js
var fs = test.promisifyAll(require('fs'));
    // just for test, otherwise for get string directly
    // use fs.readFileAsync(__filename, 'utf8')
    fs.readFileAsync(__filename)
      .when(function(raw) {
        test
          .string(raw.toString())
            .isNotEmpty()
            .contains('I m here !!!')
        ;
        return raw.toString();
      })
      .then(function(contents) {
        test
          .string(contents)
            .isNotEmpty()
            .contains('I m here also !!!')
        ;
      })
      .catch(function(err){
        test.fail('test.promise, ' + err.message);
      })
      .finally(done)
      .done()
    ;
```

test.promisify().

```js
var readFile = Promise.promisify(require('fs').readFile);
    readFile(__filename, 'utf8')
      .then(function(contents) {
        test
          .string(contents)
            .isNotEmpty()
            .contains('I m here !!!')
        ;
      })
      .catch(function(err){
        test.fail('test.promise, ' + err.message);
      })
      .finally(done)
      .done()
    ;
```

test.promise.given.

```js
var fs = test.promisifyAll(require('fs'));
    test.promise
      .given({
        a:'a value',
        b: 2,
        contents: fs.readFileAsync(__filename, 'utf8')
      })
      .when(function(given) {
        test
          .string(given.a)
            .isIdenticalTo('a value')
          .number(given.b)
            .isIdenticalTo(2)
          .bool(test.promise.is(given.contents))
            .isFalse()
          .string(given.contents)
            .isNotEmpty()
            .contains('I m here !!!')
        ;
        return given.contents;
      })
      .then(function(contents) {
        test
          .bool(test.promise.is(contents))
            .isFalse()
          .string(contents)
            .isNotEmpty()
            .contains('I m here also !!!')
        ;
      })
      .catch(function(err){
        test.fail('test.promise, ' + err.message);
      })
      .done()
    ;
    test.promise
      .given(function() {
        return 123;
      })
      .then(function(num) {
        test.number(num).isIdenticalTo(123);
      })
      .catch(function(err){
        test.fail('test.promise, ' + err.message);
      })
      .done()
    ;
    test.promise
      .given(fs.readFileAsync(__filename, 'utf8'))
      .then(function(contents) {
        test.string(contents).contains('I m here !!!');
      })
      .catch(function(err){
        test.fail('test.promise, ' + err.message);
      })
      .done()
    ;
    test.promise
      .given(function() {
        return fs.readFileAsync(__filename, 'utf8');
      })
      .then(function(contents) {
        test.string(contents).contains('I m here !!!');
      })
      .catch(function(err){
        test.fail('test.promise, ' + err.message);
      })
      .finally(done)
      .done()
    ;
```

test.promise.is.

```js
var assert = test.assert;
var is     = test.promise.is;
var fs     = test.promise.promisifyAll(require('fs'));
assert(!is({}));
assert(!is([]));
assert(!is(null));
assert(!is(0));
assert(!is(1));
assert(!is('a'));
assert(!is(function() {}));
assert(is(fs.readFileAsync(__filename, 'utf8')));
assert(is(test.promise.given([1, 2, 3])));
assert(is(test.promise.given({a: 1, b: 2})));
assert(is(test.promise.given(function() {})));
```

Async example.

```js
var fs = test.promisifyAll(require('fs'));
    fs.readFileAsync(require('path').resolve() + '/package.json')
      .when(JSON.parse)
      .then(function(pkg) {
        test
          .object(pkg)
            .hasKey('name', 'unit.js')
            .hasKey('version')
          .value(pkg.version)
            .isGreaterThan('1')
        ;
      })
      .catch(SyntaxError, function(err) {
        test.fail('Syntax error');
      })
      .catch(function(err){
        test.fail(err.message);
      })
      .done()
    ;
```

<a name="asserter-array"></a>
# Asserter array()
<a name="asserter-array-array-behavior"></a>
## array() behavior
Does not contains assertions from the assertions containers.

```js
test
  .value(test.array([]).hasHeader)
    .isUndefined()
  .value(test.array([]).isError)
    .isUndefined()
  .value(test.array([]).hasMessage)
    .isUndefined()
  .value(test.array([]).isInfinite)
    .isUndefined()
;
```

Assert that the tested value is an `array`.

```js
var Foo = function Foo(){};
        test
          .array([])
          .array(['a', 'b', 'c'])
          .array(new Array())
          .case('Test failure', function(){
            test
              .exception(function(){
                test.array({});
              })
              .exception(function(){
                test.array('Foo');
              })
              .exception(function(){
                test.array(Foo);
              })
              .exception(function(){
                test.array(1);
              })
              .exception(function(){
                test.array(undefined);
              })
              .exception(function(){
                test.array(true);
              })
              .exception(function(){
                test.array(false);
              })
              .exception(function(){
                test.array(null);
              })
              .exception(function(){
                test.array(function(){});
              })
            ;
          })
        ;
```

<a name="asserter-array-assertions-of-array"></a>
## Assertions of array()
is(expected).

```js
test
        .array(['foo', [0, 1]])
          .is(['foo', [0, 1]])
        .case('Test failure', function(){
          test
            .exception(function(){
              test.array(['foo', [0, 1]])
                .is(['foo', [0, '1']]);
            })
            .exception(function(){
              test.array(['foo', [0, 1]])
                .is(['foo', [0, 1, 2]]);
            })
            .exception(function(){
              test.array(['foo', [0, 1]])
                .is(['foo', [0]]);
            })
            .exception(function(){
              test.array(['foo', [0, 1]])
                .is(['foobar', [0, 1]]);
            })
          ;
        })
      ;
```

isNot(expected).

```js
test
        .array(['foo', [0, 1]])
          .isNot(['foo', [0, '1']])
        .exception(function(){
          test.array(['foo', [0, 1]])
            .isNot(['foo', [0, 1]]);
        })
      ;
```

isIdenticalTo(expected).

```js
var
        arr = [1],
        arr2 = arr
      ;
      test
        .array(arr)
          .isIdenticalTo(arr2)
        .exception(function(){
          test.array(arr)
            .isIdenticalTo([1]);
        })
      ;
```

isNotIdenticalTo(expected).

```js
var
        arr = [1],
        arr2 = arr
      ;
      test
        .array(arr)
          .isNotIdenticalTo([1])
        .exception(function(){
          test.array(arr)
            .isNotIdenticalTo(arr2);
        })
      ;
```

isEqualTo(expected).

```js
var
        arr = [1],
        arr2 = arr
      ;
      test
        .array(arr)
          .isEqualTo(arr2)
        .exception(function(){
          test.array(arr)
            .isEqualTo([1]);
        })
      ;
```

isNotEqualTo(expected).

```js
var
        arr = [1],
        arr2 = arr
      ;
      test
        .array(arr)
          .isNotEqualTo([1])
        .exception(function(){
          test.array(arr)
            .isNotEqualTo(arr2);
        })
      ;
```

match(expected).

```js
test
        .array(['a', 'b', 'c'])
          .match(/[a-z]/)
        .array([42, 10])
          .match(function(actual) {
            return actual[1] === 10;
          })
        .exception(function() {
          test.array([42, '10']).match(function(actual) {
            return actual[1] === 10;
          });
        })
      ;
```

notMatch(expected).

```js
test
        .array(['a', 'b', 'c'])
          .notMatch(/[d-z]/)
        .array([42, 10])
          .notMatch(function(actual) {
            return actual[1] === '10';
        })
        .exception(function() {
          test.array([42, '10']).notMatch(function(actual) {
            return actual[0] === 42;
          });
        })
      ;
```

isValid(expected).

```js
test
        .array(['a', 'b', 'c'])
          .isValid(/[a-z]/)
        .array([42, 10])
          .isValid(function(actual) {
            return actual[1] === 10;
          })
        .exception(function() {
          test.array([42, '10']).isValid(function(actual) {
            return actual[1] === 10;
          });
        })
      ;
```

isNotValid(expected).

```js
test
        .array(['a', 'b', 'c'])
          .isNotValid(/[d-z]/)
        .array([42, 10])
          .isNotValid(function(actual) {
            return actual[1] === '10';
        })
        .exception(function() {
          test.array([42, '10']).isNotValid(function(actual) {
            return actual[0] === 42;
          });
        })
      ;
```

matchEach(expected).

```js
test
        .array([10, 11, 12])
          .matchEach(function(it) {
            return it >= 10;
          })
        .exception(function() {
          // error if one or several does not match
          test.array([10, 11, 12]).matchEach(function(it) {
            return it >= 11;
          });
        })
      ;
```

notMatchEach(expected).

```js
test
        .array([10, 11, 12])
          .notMatchEach(function(it) {
            return it >= 13;
          })
        .exception(function() {
          // error all match
          test.array([10, 11, 12]).notMatchEach(function(it) {
            return it >= 11;
          });
        })
      ;
```

isEmpty().

```js
test
        .array([])
          .isEmpty()
        .exception(function(){
          test.array([0])
            .isEmpty();
        })
        .exception(function(){
          test.array([''])
            .isEmpty();
        })
      ;
```

isNotEmpty().

```js
test
        .array(['a'])
          .isNotEmpty()
        .exception(function(){
          test.array([])
            .isNotEmpty();
        })
      ;
```

hasLength(expected).

```js
test
  .array([1, 2])
    .hasLength(2)
  .exception(function(){
    test.array([1, 2])
      .hasLength(1);
  })
;
```

hasNotLength(expected).

```js
test
  .array([1, 2])
    .hasNotLength(1)
  .exception(function(){
    test.array([1, 2])
      .hasNotLength(2);
  })
;
```

isEnumerable(property).

```js
var arr = ['is enumerable'];
test
  .array(arr)
    .isEnumerable(0)
  .array(arr)
    .isNotEnumerable('length')
  .exception(function(){
    test.array(arr)
      .isEnumerable('length');
  })
;
```

isNotEnumerable(property).

```js
var arr = ['is enumerable'];
test
  .array(arr)
    .isNotEnumerable('length')
  .array(arr)
    .isEnumerable(0)
  .exception(function(){
    test.array(arr)
      .isNotEnumerable(0);
  })
;
```

hasProperty(property [, value]).

```js
test
  .array(['a', 'b'])
    .hasProperty(1)
    .hasProperty(0, 'a')
  .exception(function(){
    test.array(['a', 'b'])
      .hasProperty(3);
  })
  .exception(function(){
    test.array(['a', 'b'])
      .hasProperty(0, 'b');
  })
;
```

hasNotProperty(property [, value]).

```js
test
  .array(['a', 'b'])
    .hasNotProperty(2)
    .hasNotProperty(0, 'b')
  .exception(function(){
    test.array(['a', 'b'])
      .hasNotProperty(0);
  })
  .exception(function(){
    test.array(['a', 'b'])
      .hasNotProperty(1, 'b');
  })
;
```

hasKey(key [, value]).

```js
test
  .array(['a', 'b'])
    .hasKey(1)
    .hasKey(0, 'a')
  .exception(function(){
    test.array(['a', 'b'])
      .hasKey(3);
  })
  .exception(function(){
    test.array(['a', 'b'])
      .hasKey(0, 'b');
  })
;
```

notHasKey(key [, value]).

```js
test
  .array(['a', 'b'])
    .notHasKey(2)
    .notHasKey(0, 'b')
  .exception(function(){
    test.array(['a', 'b'])
      .notHasKey(0);
  })
  .exception(function(){
    test.array(['a', 'b'])
      .notHasKey(1, 'b');
  })
;
```

hasValue(expected).

```js
test
  .array([1, 42, 3])
    .hasValue(42)
  .exception(function(){
    test.array([1, 42, 3])
      .hasValue(0);
  })
;
```

notHasValue(expected).

```js
test
  .array([1, 42, 3])
    .notHasValue(4)
  .exception(function(){
    test.array([1, 42, 3])
      .notHasValue(42);
  })
;
```

hasValues(expected).

```js
test
  .array([1, 42, 3])
    .hasValues([42, 3])
  .exception(function(){
    test.array([1, 42, 3])
      .hasValues([42, 3, 10]);
  })
;
```

notHasValues(expected).

```js
test
  .array([1, 42, 3])
    .notHasValues([4, 2])
  .exception(function(){
    test.array([1, 42, 3])
      .notHasValues([4, 1]);
  })
;
```

contains(expected [, ...]).

```js
test
  .array([1,2,3])
    .contains([3])
  .array([1,2,3])
    .contains([1, 3])
  .array([1,2,3])
    .contains([3], [1, 3])
  .array([1, 2, 3, { a: { b: { d: 12 }}}])
    .contains([2], [1, 2], [{ a: { b: {d: 12}}}])
  .array([[1],[2],[3]])
    .contains([[3]])
  .array([[1],[2],[3, 4]])
    .contains([[3]])
  .array([{a: 'a'}, {b: 'b', c: 'c'}])
    .contains([{a: 'a'}], [{b: 'b'}])
  .exception(function(){
    test.array([1,2,3])
      .contains([0]);
  })
;
```

notContains(expected [, ...]).

```js
test
  .array([[1],[2],[3, 4]])
    .notContains([[0]])
  .array([{a: 'a'}, {b: 'b', c: 'c'}])
    .notContains([{a: 'b'}], [{c: 'b'}])
  .exception(function(){
    test.array([{a: 'a'}, {b: 'b', c: 'c'}])
    .notContains([{a: 'a'}], [{b: 'b'}]);
  })
;
```

isReverseOf(expected).

```js
test
        .array([1, 2, 3])
          .isReverseOf([3, 2, 1])
        .exception(function() {
          test.array([1, 2, 3])
            .isReverseOf([1, 2, 3]);
        })
        .exception(function() {
          test.array([1, 2, 3])
            .isReverseOf([3, 2, 2, 1]);
        })
      ;
```

isNotReverseOf(expected).

```js
test
        .array([1, 2, 2, 3])
          .isNotReverseOf([3, 2, 1])
        .exception(function() {
          test.array([3, 2, 1])
            .isNotReverseOf([3, 2, 1]);
        })
      ;
```

<a name="asserter-bool"></a>
# Asserter bool()
<a name="asserter-bool-bool-behavior"></a>
## bool() behavior
Does not contains assertions from the assertions containers.

```js
test
        .value(test.bool(true).hasHeader)
          .isUndefined()
        .value(test.bool(true).hasProperty)
          .isUndefined()
        .value(test.bool(true).hasMessage)
          .isUndefined()
      ;
```

Assert that the tested value is a `boolean`.

```js
test
        .bool(true)
        .bool(false)
        .case('Test failure', function(){
          test
            .exception(function(){
              test.bool();
            })
            .exception(function(){
              test.bool(0);
            })
            .exception(function(){
              test.bool(1);
            })
            .exception(function(){
              test.bool(undefined);
            })
            .exception(function(){
              test.bool(null);
            })
            .exception(function(){
              test.bool('');
            })
            .exception(function(){
              test.bool('true');
            })
            .exception(function(){
              test.bool('false');
            })
            .exception(function(){
              test.bool('1');
            })
            .exception(function(){
              test.bool('0');
            })
            .exception(function(){
              test.bool([]);
            })
            .exception(function(){
              test.bool({});
            })
            .exception(function(){
              test.bool(new Boolean('false')).isFalse(); // object
            })
            .exception(function(){
              test.bool(Boolean('false')).isFalse();
            })
          ;
        })
      ;
```

<a name="asserter-bool-assertions-of-bool"></a>
## Assertions of bool()
isTrue().

```js
test
        .bool(true)
          .isTrue()
        .exception(function(){
          test.bool(false).isTrue();
        })
      ;
```

isNotTrue().

```js
test
        .bool(false)
          .isNotTrue()
        .exception(function(){
          test.bool(true).isNotTrue();
        })
      ;
```

isFalse().

```js
test
        .bool(false)
          .isFalse()
        .exception(function(){
          test.bool(true).isFalse();
        })
      ;
```

isNotFalse().

```js
test
        .bool(true)
          .isNotFalse()
        .exception(function(){
          test.bool(false).isNotFalse();
        })
      ;
```

<a name="asserter-date"></a>
# Asserter date()
<a name="asserter-date-date-behavior"></a>
## date() behavior
Does not contains assertions from the assertions containers.

```js
test
        .value(test.date(new Date()).hasHeader)
          .isUndefined()
        .value(test.date(new Date()).hasProperty)
          .isUndefined()
        .value(test.date(new Date()).hasMessage)
          .isUndefined()
        .value(test.date(new Date()).isInfinite)
          .isUndefined()
      ;
```

Assert that the tested value is an instance of `Date`.

```js
test
        .date(new Date())
        .date(new Date('2010, 5, 20'))
        .case('Test failure', function(){
          test
            .exception(function(){
              test.date('2010 5 20');
            })
            .exception(function(){
              test.date(2010);
            })
            .exception(function(){
              test.date(Date);
            })
          ;
        })
      ;
```

<a name="asserter-date-assertions-of-date"></a>
## Assertions of date()
is(expected).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .is(new Date('2010, 5, 20'))
        .case('Test failure', function(){
          test
            .exception(function(){
              test.date(date).is(/2010/);
            })
            .exception(function(){
              test.date(date).is(new Date('2011, 5, 20'));
            })
          ;
        })
      ;
```

isNot(expected).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .isNot(new Date('2012, 02, 28'))
        .case('Test failure', function(){
          test
            .exception(function(){
              test.date(date).isNot(new Date('2010, 5, 20'));
            })
            .exception(function(){
              test.date(date).isNot(date);
            })
          ;
        })
      ;
```

isIdenticalTo(expected).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .isIdenticalTo(date)
        .exception(function(){
          test.date(date).isIdenticalTo(new Date('2010, 5, 20'));
        })
      ;
```

isNotIdenticalTo(expected).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .isNotIdenticalTo(new Date('2010, 5, 20'))
        .exception(function(){
          test.date(date).isNotIdenticalTo(date);
        })
      ;
```

isEqualTo(expected).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .isEqualTo(date)
        .exception(function(){
          test.date(date).isEqualTo(new Date('2010, 5, 20'));
        })
      ;
```

isNotEqualTo(expected).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .isNotEqualTo(new Date('2010, 5, 20'))
        .exception(function(){
          test.date(date).isNotEqualTo(date);
        })
      ;
```

match(expected).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .match(/\d/)
        .exception(function(){
          test.date(date).match(/^\d$/);
        })
      ;
```

notMatch(expected).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .notMatch(/^\d$/)
        .exception(function(){
          test.date(date).notMatch(/\d/);
        })
      ;
```

isValid(expected).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .isValid(/\d/)
        .exception(function(){
          test.date(date).isValid(/^\d$/);
        })
      ;
```

isNotValid(expected).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .isNotValid(/^\d$/)
        .exception(function(){
          test.date(date).isNotValid(/\d/);
        })
      ;
```

isBetween(begin, end).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .isBetween(new Date('1982, 02, 17'), new Date('2012, 02, 28'))
        .exception(function(){
          test.date(date).isBetween(
            new Date('2012, 02, 28'), new Date('1982, 02, 17')
          );
        })
      ;
```

isNotBetween(begin, end).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .isNotBetween(new Date('2011, 02, 17'), new Date('2012, 02, 28'))
        .exception(function(){
          test.date(date).isNotBetween(
            new Date('1982, 02, 17'), new Date('2012, 02, 28')
          );
        })
      ;
```

isBefore(expected).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .isBefore(new Date('2012, 02, 28'))
        .exception(function(){
          test.date(date).isBefore(new Date('1982, 02, 17'));
        })
      ;
```

isAfter(expected).

```js
var date = new Date('2010, 5, 20');
      test
        .date(date)
          .isAfter(new Date('1982, 02, 17'))
        .exception(function(){
          test.date(date).isAfter(new Date('2012, 02, 28'));
        })
      ;
```

<a name="asserter-error"></a>
# Asserter error()
<a name="asserter-error-error-behavior"></a>
## error() behavior
Does not contains inappropriate assertions from the assertions containers.

```js
var
        indicator = 0,
        trigger = function(){
          throw new Error('Whoops!');
        },
        deletedAssertions = [
          // types
          'isType', 'isNotType', 'isObject', 'isArray', 'isString',
          'isNumber', 'isBool', 'isBoolean', 'isNull', 'isUndefined',
          // types augmented
          'isRegExp', 'isNotRegExp', 'isDate', 'isNotDate', 'isArguments',
          'isNotArguments', 'isEmpty', 'isNotEmpty',
          // quantification
          'hasLength', 'hasNotLength',
          // containers
          'hasProperties', 'hasNotProperties', 'hasOwnProperties',
          'hasKeys', 'notHasKeys',
          'hasValue', 'notHasValue', 'hasValues', 'notHasValues',
          'contains', 'notContains',
          // string
          'startsWith', 'notStartsWith', 'endsWith', 'notEndsWith'
        ]
      ;
      // global
      test
        .value(test.error(trigger).hasHeader)
          .isUndefined()
        .value(test.error(trigger).isInfinite)
          .isUndefined()
      ;
      // inherited from exception
      deletedAssertions.map(function(method){
        test.value(test.error(trigger)[method]).isUndefined();
        indicator++;
      });
      // out of trigger block,
      // beacause test.error(trigger) throws the trigger in the block
      var error = test.error(trigger);
      // ensures the test method
      test.exception(function(){
        test.value(error['hasMessage']).isUndefined();
      });
      test.number(indicator).isIdenticalTo(deletedAssertions.length);
```

Fails if the tested exception is not an instance of Error.

```js
var
        allCatch = {number: true, string: true, object: true, array: true},
        status = {number: false, string: false, object: false, array: false},
        check = function(name, trigger){
          try{
            test.error(trigger);
          }catch(e){
            status[name] = true;
          }
        },
        error = function(){
          throw new Error('foo');
        },
        number = function(){
          throw 1;
        },
        string = function(){
          throw 'Error';
        },
        object = function(){
          throw {name: Error, message: 'Whoops!'};
        },
        array = function(){
          throw [Error];
        }
      ;
      test
        .error(error)
        .when(function(){
          check('number', number);  // status[name] = true;
          check('string', string);  // status[name] = true;
          check('object', object);  // status[name] = true;
          check('array', array);    // status[name] = true;
        })
        .object(status)
          .is(allCatch)
      ;
      // for comparing with exception
      var
        allCatch = {number: true, string: true, object: true, array: true},
        status = {number: false, string: false, object: false, array: false},
        check = function(name, trigger){
          try{
            test.exception(trigger);
          }catch(e){
            status[name] = true;
          }
        }
      ;
      test
        .exception(error)
          .isError()
        .when(function(){
          check('number', number);  // not thrown
          check('string', string);  // not thrown
          check('object', object);  // not thrown
          check('array', array);    // not thrown
        })
        .object(status)
          .isNot(allCatch)
          .is({number: false, string: false, object: false, array: false})
      ;
```

dependency injection.

```js
var spy = test.spy();
      test.$di.set('spyException', spy);
      test
        .error(function() {
          this.spyException('arg1', 'arg2');
          throw new Error('Whoops!');
        })
        .bool(spy.calledOnce && spy.calledWithExactly('arg1', 'arg2'))
          .isTrue()
      ;
```

<a name="asserter-error-assertions-of-error"></a>
## Assertions of error()
is(expected).

```js
var error = new Error('Whoops !');
      var trigger = function(){
        throw error;
      };
      test
        .error(trigger)
          .is(error)
          .is(new Error('Whoops !'))
        .case('Test failure', function(){
          test
            .value(function(){
              test.error(trigger).is({message: 'Whoops !'});
            })
            .throws()
            .value(function(){
              test.error(trigger).is(new TypeError('Whoops !'));
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

isNot(expected).

```js
var error = new Error('Whoops !');
var trigger = function(){
  throw error;
};
test
  .error(trigger)
    .isNot({message: 'Whoops !'})
  // Test failure
  .value(function(){
    test.error(trigger).isNot(error);
  })
  .throws()
;
```

isIdenticalTo(expected).

```js
var error = new Error('Whoops !');
      var trigger = function(){
        throw error;
      };
      test
        .error(trigger)
          .isIdenticalTo(error)
        // Test failure
        .value(function(){
          test.error(trigger).isIdenticalTo(new Error('Whoops !'));
        })
        .throws()
      ;
```

isNotIdenticalTo(expected).

```js
var error = new Error('Whoops !');
      var trigger = function(){
        throw error;
      };
      test
        .error(trigger)
          .isNotIdenticalTo(new Error('Whoops !'))
        // Test failure
        .value(function(){
          test.error(trigger).isNotIdenticalTo(error);
        })
        .throws()
      ;
```

isEqualTo(expected).

```js
var error = new Error('Whoops !');
      var trigger = function(){
        throw error;
      };
      test
        .error(trigger)
          .isEqualTo(error)
        // Test failure
        .value(function(){
          test.error(trigger).isEqualTo(new Error('Whoops !'));
        })
        .throws()
      ;
```

isNotEqualTo(expected).

```js
var error = new Error('Whoops !');
      var trigger = function(){
        throw error;
      };
      test
        .error(trigger)
          .isNotEqualTo(new Error('Whoops !'))
        // Test failure
        .value(function(){
          test.error(trigger).isNotEqualTo(error);
        })
        .throws()
      ;
```

match(expected).

```js
var
        // create an indicator for monitoring the example and the test
        indicator = test.createCollection(),
        trigger = function(){
          indicator.set('error trigger called', true);
          throw new Error('Whoops!');
        }
      ;
      test
        .error(trigger)
          .match('Whoops!')
          .match(/Whoops/)
          .match(function(exception){
            indicator.set('custom error validation called', true);
            return (exception instanceof Error) && /whoops/i.test(exception);
          })
        // just for the example and for the test
        .bool(indicator.get('error trigger called')).isTrue()
        .bool(indicator.get('custom error validation called')).isTrue()
        .case('Test failure', function(){
          test
            .value(function(){
              test.error(trigger).match('Hey');
            })
            .throws()
            .value(function(){
              test.error(trigger).match(/Hey/);
            })
            .throws()
            .value(function(){
              test.error(trigger).match(function(error){
                return error instanceof RegExp;
              });
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

notMatch(expected).

```js
var
        // create an indicator for monitoring the example and the test
        indicator = test.createCollection(),
        trigger = function(){
          indicator.set('error trigger called', true);
          throw new Error('Whoops!');
        }
      ;
      test
        .error(trigger)
          .notMatch('Yeah an error')
          .notMatch(/Yeah/)
          .notMatch(function(exception){
            indicator.set('custom error validation called', true);
            return /yeah/.test(exception);
          })
        // just for the example and for the test
        .bool(indicator.get('error trigger called')).isTrue()
        .bool(indicator.get('custom error validation called')).isTrue()
        .case('Test failure', function(){
          test
            .value(function(){
              test.error(trigger).notMatch('Whoops!');
            })
            .throws()
            .value(function(){
              test.error(trigger).notMatch(/Whoops/);
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

isValid(expected).

```js
var
        // create an indicator for monitoring the example and the test
        indicator = test.createCollection(),
        trigger = function(){
          indicator.set('error trigger called', true);
          throw new Error('Whoops!');
        }
      ;
      test
        .error(trigger)
          .isValid('Whoops!')
          .isValid(/Whoops/)
          .isValid(function(exception){
            indicator.set('custom error validation called', true);
            return (exception instanceof Error) && /whoops/i.test(exception);
          })
        // just for the example and for the test
        .bool(indicator.get('error trigger called')).isTrue()
        .bool(indicator.get('custom error validation called')).isTrue()
        .case('Test failure', function(){
          test
            .value(function(){
              test.error(trigger).isValid('Hey');
            })
            .throws()
            .value(function(){
              test.error(trigger).isValid(/Hey/);
            })
            .throws()
            .value(function(){
              test.error(trigger).isValid(function(error){
                return error instanceof RegExp;
              });
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

isNotValid(expected).

```js
var
        // create an indicator for monitoring the example and the test
        indicator = test.createCollection(),
        trigger = function(){
          indicator.set('error trigger called', true);
          throw new Error('Whoops!');
        }
      ;
      test
        .error(trigger)
          .isNotValid('Yeah an error')
          .isNotValid(/Yeah/)
          .isNotValid(function(exception){
            indicator.set('custom error validation called', true);
            return /yeah/.test(exception);
          })
        // just for the example and for the test
        .bool(indicator.get('error trigger called')).isTrue()
        .bool(indicator.get('custom error validation called')).isTrue()
        .case('Test failure', function(){
          test
            .value(function(){
              test.error(trigger).isNotValid('Whoops!');
            })
            .throws()
            .value(function(){
              test.error(trigger).isNotValid(/Whoops/);
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

isEnumerable(property).

```js
var error = new Error('Whoops !');
      error.foo = 'bar';
      test
        .error(function(){
          throw error;
        })
        .isEnumerable('foo')
        // Test failure
        .value(function(){
          test
            .error(function(){
              throw error;
            })
            .isEnumerable('message')
          ;
        })
        .throws()
      ;
```

isNotEnumerable(property).

```js
var error = new Error('Whoops !');
      error.foo = 'bar';
      test
        .error(function(){
          throw error;
        })
        .isNotEnumerable('message')
        // Test failure
        .value(function(){
          test
            .error(function(){
              throw error;
            })
            .isNotEnumerable('foo')
          ;
        })
        .throws()
      ;
```

isFrozen().

```js
var
        error = new Error('Whoops !'),
        frozenError = new Error('Whoops !')
      ;
      Object.freeze(frozenError);
      test
        .error(function(){
          throw frozenError;
        })
        .isFrozen()
        // Test failure
        .value(function(){
          test
            .error(function(){
              throw error;
            })
            .isFrozen()
          ;
        })
        .throws()
      ;
```

isNotFrozen().

```js
var
        error = new Error('Whoops !'),
        frozenError = new Error('Whoops !')
      ;
      Object.freeze(frozenError);
      test
        .error(function(){
          throw error;
        })
        .isNotFrozen()
        // Test failure
        .value(function(){
          test
            .error(function(){
              throw frozenError;
            })
            .isNotFrozen()
          ;
        })
        .throws()
      ;
```

isInstanceOf(expected).

```js
test
        .error(function(){
          throw new TypeError('Whoops !');
        })
        .isInstanceOf(TypeError)
        // Test failure
        .value(function(){
          test
            .error(function(){
              throw new Error('Bad type');
            })
            .isInstanceOf(TypeError)
          ;
        })
        .throws()
      ;
```

isNotInstanceOf(expected).

```js
test
        .error(function(){
          throw new Error('Whoops !');
        })
        .isNotInstanceOf(TypeError)
        // Test failure
        .value(function(){
          test
            .error(function(){
              throw new TypeError('Bad type');
            })
            .isNotInstanceOf(TypeError)
          ;
        })
        .throws()
      ;
```

hasProperty(property [, value]).

```js
test
        .error(function(){
          throw new Error('Whoops !');
        })
        .hasProperty('message')
        .hasProperty('message', 'Whoops !')
        .hasProperty('constructor')
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .hasProperty('foo')
              ;
            })
            .throws()
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .hasProperty('message', 'whoops')
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasNotProperty(property [, value]).

```js
test
        .error(function(){
          throw new Error('Whoops !');
        })
        .hasNotProperty('foo')
        .hasNotProperty('message', 'whoops')
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .hasNotProperty('message')
              ;
            })
            .throws()
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .hasNotProperty('message', 'Whoops !')
              ;
            })
            .throws()
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .hasNotProperty('constructor')
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasOwnProperty(property [, value]).

```js
test
        .error(function(){
          throw new Error('Whoops !');
        })
        .hasOwnProperty('message')
        .hasOwnProperty('message', 'Whoops !')
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .hasOwnProperty('foo')
              ;
            })
            .throws()
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .hasOwnProperty('message', 'Grrrr !')
              ;
            })
            .throws()
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .hasOwnProperty('constructor')
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasNotOwnProperty(property [, value]).

```js
test
        .error(function(){
          throw new Error('Whoops !');
        })
        .hasNotOwnProperty('foo')
        .hasNotOwnProperty('message', 'Grrrr !')
        .hasNotOwnProperty('constructor')
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .hasNotOwnProperty('message')
              ;
            })
            .throws()
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .hasNotOwnProperty('message', 'Whoops !')
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasKey(key [, value]).

```js
test
        .error(function(){
          throw new Error('Whoops !');
        })
        .hasKey('message')
        .hasKey('message', 'Whoops !')
        .hasKey('constructor')
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .hasKey('foo')
              ;
            })
            .throws()
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .hasKey('message', 'whoops')
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

notHasKey(key [, value]).

```js
test
        .error(function(){
          throw new Error('Whoops !');
        })
        .notHasKey('foo')
        .notHasKey('message', 'whoops')
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .notHasKey('message')
              ;
            })
            .throws()
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .notHasKey('message', 'Whoops !')
              ;
            })
            .throws()
            .value(function(){
              test
                .error(function(){
                  throw new Error('Whoops !');
                })
                .notHasKey('constructor')
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasMessage(expected).

```js
var
        // create an indicator for monitoring the example and the test
        indicator = test.createCollection(),
        trigger = function(){
          indicator.set('error constructor called', true);
          throw new Error("I'm a ninja !");
        },
        resetIndicator = function(){
          // empty
          indicator.setAll({});
        },
        exception
      ;
      test
        .error(trigger)
          .hasMessage("I'm a ninja !")
        // just for the example and for the test
        .bool(indicator.get('error constructor called')).isTrue()
        // reset indicator
        // 'then' does nothing, is just pass-through method for a fluent chain.
        .then(resetIndicator())
          .error(trigger)
            .hasMessage(/ninja/)
          // just for the example and for the test
          .bool(indicator.get('error constructor called')).isTrue()
        // reset indicator
        .then(resetIndicator())
        // out of trigger block,
        // because test.error(trigger) throws the trigger in the block
        // also you can use test.value().throws() (see after)
        .case(exception = test.error(trigger))
          .error(function(){
            indicator.set('ninjaa is not in error message', true);
            // fails because ninjaa is not in error message
            exception.hasMessage(/ninjaa/);
          })
          // just for the example and for the test
          .bool(indicator.get('error constructor called')).isTrue()
          .bool(indicator.get('ninjaa is not in error message')).isTrue()
        // the above example is equal to
        .then(resetIndicator())
          .value(function(){
            indicator.set('ninjaa is not in error message', true);
            // fails because ninjaa is not in error message
            test.error(trigger).hasMessage(/ninjaa/);
          })
          .throws()
          // just for the example and for the test
          .bool(indicator.get('error constructor called')).isTrue()
          .bool(indicator.get('ninjaa is not in error message')).isTrue()
      ;
```

<a name="asserter-exception"></a>
# Asserter exception()
<a name="asserter-exception-exception-behavior"></a>
## exception() behavior
Does not contains assertions from the assertions containers.

```js
test
        .value(test.exception(function(){ throw new Error('hu'); }).hasHeader)
          .isUndefined()
        .value(test.exception(function(){ throw new Error('hu'); }).isBetween)
          .isUndefined()
```

Takes a function that will throws an exception.

```js
var
        indicator,
        trigger = function(){
          indicator = true;
          throw new Error("I'm a ninja !");
        }
      ;
      // Apply the trigger and assert that an exception is thrown
      test
        .exception(trigger)
        // just for the example and for the test
        .bool(indicator).isTrue()
        .given(indicator = false)
          .exception(function(){
            indicator = true;
            throw new Error('Whoops!');
          })
          // just for the example and for the test
          .bool(indicator).isTrue()
      ;
```

Error if the trigger don't throws an exception.

```js
var indicator;
      test
        .given(indicator = false)
          .value(function(){
            // no error thrown by the trigger,
            // then throw 'Error: Missing expected exception'
            test.exception(function(){
              indicator = true;
            });
          })
          .throws()
          // just for the example and for the test
          .bool(indicator).isTrue()
      ;
```

Assert that thrown with the Error class and a given message.

```js
var
        // create an indicator for monitoring the example and the test
        indicator = test.createCollection(),
        fn = function(){
          indicator.set('error constructor called', true);
          throw new Error("I'm a ninja !");
        },
        resetIndicator = function(){
          // empty
          indicator.setAll({});
        },
        exception
      ;
      test
        .exception(fn)
          .isError()
          .hasMessage("I'm a ninja !")
        // just for the example and for the test
        .bool(indicator.get('error constructor called')).isTrue()
        .when(exception = test.exception(fn))
          .exception(function(){
            // fails because is not the error message
            exception.isError().hasMessage("I'm a not ninja !");
          })
        .given(resetIndicator())
          // Assert that thrown with the Error class
          // and with the message (regExp)
          .exception(fn)
            .isError()
            .hasMessage(/ninja/)
          // just for the example and for the test
          .bool(indicator.get('error constructor called')).isTrue()
          .when(exception = test.exception(fn))
            .exception(function(){
              // fails because 'ninjaa' is not in error message
              exception.isError().hasMessage(/ninjaa/);
            })
        .given(resetIndicator())
          .exception(function(){
            indicator.set('Whoops error, is called', true);
            throw new Error('Whoops!');
          })
          .hasMessage('Whoops!')
          .isInstanceOf(Error)
          // just for the example and for the test
          .bool(indicator.get('Whoops error, is called')).isTrue()
      ;
```

dependency injection.

```js
var spy = test.spy();
      test.$di.set('spyException', spy);
      test
        .exception(function() {
          this.spyException('arg1', 'arg2');
          throw new Error('Whoops!');
        })
        .bool(spy.calledOnce && spy.calledWithExactly('arg1', 'arg2'))
          .isTrue()
      ;
```

<a name="asserter-exception-assertions-of-exception"></a>
## Assertions of exception()
is(expected).

```js
var error = new Error('Whoops !');
      var trigger = function(){
        throw error;
      };
      test
        .exception(trigger)
          .is(error)
          .is(new Error('Whoops !'))
        .case('Test failure', function(){
          test
            .value(function(){
              test.exception(trigger).is({message: 'Whoops !'});
            })
            .throws()
            .value(function(){
              test.exception(trigger).is(new String('Whoops !'));
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

isNot(expected).

```js
var error = new Error('Whoops !');
var trigger = function(){
  throw error;
};
test
  .exception(trigger)
    .isNot({message: 'Whoops !'})
  // Test failure
  .value(function(){
    test.exception(trigger).isNot(error);
  })
  .throws()
;
```

isIdenticalTo(expected).

```js
var error = new Error('Whoops !');
      var trigger = function(){
        throw error;
      };
      test
        .exception(trigger)
          .isIdenticalTo(error)
        // Test failure
        .value(function(){
          test.exception(trigger).isIdenticalTo(new Error('Whoops !'));
        })
        .throws()
      ;
```

isNotIdenticalTo(expected).

```js
var error = new Error('Whoops !');
      var trigger = function(){
        throw error;
      };
      test
        .exception(trigger)
          .isNotIdenticalTo(new Error('Whoops !'))
        // Test failure
        .value(function(){
          test.exception(trigger).isNotIdenticalTo(error);
        })
        .throws()
      ;
```

isEqualTo(expected).

```js
var error = new Error('Whoops !');
      var trigger = function(){
        throw error;
      };
      test
        .exception(trigger)
          .isEqualTo(error)
        // Test failure
        .value(function(){
          test.exception(trigger).isEqualTo(new Error('Whoops !'));
        })
        .throws()
      ;
```

isNotEqualTo(expected).

```js
var error = new Error('Whoops !');
      var trigger = function(){
        throw error;
      };
      test
        .exception(trigger)
          .isNotEqualTo(new Error('Whoops !'))
        // Test failure
        .value(function(){
          test.exception(trigger).isNotEqualTo(error);
        })
        .throws()
      ;
```

match(expected).

```js
var
        // create an indicator for monitoring the example and the test
        indicator = test.createCollection(),
        trigger = function(){
          indicator.set('error trigger called', true);
          throw new Error('Whoops!');
        }
      ;
      test
        .exception(trigger)
          .match('Whoops!')
          .match(/Whoops/)
          .match(function(exception){
            indicator.set('custom error validation called', true);
            return (exception instanceof Error) && /whoops/i.test(exception);
          })
        // just for the example and for the test
        .bool(indicator.get('error trigger called')).isTrue()
        .bool(indicator.get('custom error validation called')).isTrue()
        .case('Test failure', function(){
          test
            .value(function(){
              test.exception(trigger).match('Hey');
            })
            .throws()
            .value(function(){
              test.exception(trigger).match(/Hey/);
            })
            .throws()
            .value(function(){
              test.exception(trigger).match(function(error){
                return error instanceof RegExp;
              });
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

notMatch(expected).

```js
var
        // create an indicator for monitoring the example and the test
        indicator = test.createCollection(),
        trigger = function(){
          indicator.set('error trigger called', true);
          throw new Error('Whoops!');
        }
      ;
      test
        .exception(trigger)
          .notMatch('Yeah an error')
          .notMatch(/Yeah/)
          .notMatch(function(exception){
            indicator.set('custom error validation called', true);
            return /yeah/.test(exception);
          })
        // just for the example and for the test
        .bool(indicator.get('error trigger called')).isTrue()
        .bool(indicator.get('custom error validation called')).isTrue()
        .case('Test failure', function(){
          test
            .value(function(){
              test.exception(trigger).notMatch('Whoops!');
            })
            .throws()
            .value(function(){
              test.exception(trigger).notMatch(/Whoops/);
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

isValid(expected).

```js
var
        // create an indicator for monitoring the example and the test
        indicator = test.createCollection(),
        trigger = function(){
          indicator.set('error trigger called', true);
          throw new Error('Whoops!');
        }
      ;
      test
        .exception(trigger)
          .isValid('Whoops!')
          .isValid(/Whoops/)
          .isValid(function(exception){
            indicator.set('custom error validation called', true);
            return (exception instanceof Error) && /whoops/i.test(exception);
          })
        // just for the example and for the test
        .bool(indicator.get('error trigger called')).isTrue()
        .bool(indicator.get('custom error validation called')).isTrue()
        .case('Test failure', function(){
          test
            .value(function(){
              test.exception(trigger).isValid('Hey');
            })
            .throws()
            .value(function(){
              test.exception(trigger).isValid(/Hey/);
            })
            .throws()
            .value(function(){
              test.exception(trigger).isValid(function(error){
                return error instanceof RegExp;
              });
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

isNotValid(expected).

```js
var
        // create an indicator for monitoring the example and the test
        indicator = test.createCollection(),
        trigger = function(){
          indicator.set('error trigger called', true);
          throw new Error('Whoops!');
        }
      ;
      test
        .exception(trigger)
          .isNotValid('Yeah an error')
          .isNotValid(/Yeah/)
          .isNotValid(function(exception){
            indicator.set('custom error validation called', true);
            return /yeah/.test(exception);
          })
        // just for the example and for the test
        .bool(indicator.get('error trigger called')).isTrue()
        .bool(indicator.get('custom error validation called')).isTrue()
        .case('Test failure', function(){
          test
            .value(function(){
              test.exception(trigger).isNotValid('Whoops!');
            })
            .throws()
            .value(function(){
              test.exception(trigger).isNotValid(/Whoops/);
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

isType(expected).

```js
var trigger = function(){
        throw new Error('Whoops !');
      };
      test
        .exception(trigger)
          .isType('object')
        .exception(function(){
          throw 'Whoops !';
        })
        .isType('string')
        // Test failure
        .value(function(){
          test.exception(trigger).isType('function');
        })
        .throws()
      ;
```

isNotType(expected).

```js
var trigger = function(){
        throw new Error('Whoops !');
      };
      test
        .exception(trigger)
          .isNotType('string')
        .exception(function(){
          throw 'Whoops !';
        })
        .isNotType('object')
        // Test failure
        .value(function(){
          test.exception(trigger).isNotType('object');
        })
        .throws()
      ;
```

isObject().

```js
test
        .exception(function(){
          throw new Error('Whoops !');
        })
        .isObject()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw 'error';
            })
            .isObject()
          ;
        })
        .throws()
      ;
```

isArray().

```js
test
        .exception(function(){
          throw ['error'];
        })
        .isArray()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw new Error('Whoops !');
            })
            .isArray()
          ;
        })
        .throws()
      ;
```

isString().

```js
test
        .exception(function(){
          throw 'error';
        })
        .isString()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw new Error('Whoops !');
            })
            .isString()
          ;
        })
        .throws()
      ;
```

isNumber().

```js
test
        .exception(function(){
          throw 0;
        })
        .isNumber()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw '0';
            })
            .isNumber()
          ;
        })
        .throws()
      ;
```

isBool().

```js
test
        .exception(function(){
          throw false;
        })
        .isBool()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw 0;
            })
            .isBool()
          ;
        })
        .throws()
      ;
```

isBoolean().

```js
test
        .exception(function(){
          throw true;
        })
        .isBoolean()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw 1;
            })
            .isBoolean()
          ;
        })
        .throws()
      ;
```

isNull().

```js
test
        .exception(function(){
          throw null;
        })
        .isNull()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw 0;
            })
            .isNull()
          ;
        })
        .throws()
      ;
```

isUndefined().

```js
test
        .exception(function(){
          throw undefined;
        })
        .isUndefined()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw 0;
            })
            .isUndefined()
          ;
        })
        .throws()
      ;
```

isRegExp().

```js
test
        .exception(function(){
          throw new RegExp('whoops');
        })
        .isRegExp()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw new Error('Whoops !');
            })
            .isRegExp()
          ;
        })
        .throws()
      ;
```

isNotRegExp().

```js
test
        .exception(function(){
          throw new Error('Whoops !');
        })
        .isNotRegExp()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw new RegExp('whoops');
            })
            .isNotRegExp()
          ;
        })
        .throws()
      ;
```

isDate().

```js
test
        .exception(function(){
          throw new Date();
        })
        .isDate()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw new Error('Whoops !');
            })
            .isDate()
          ;
        })
        .throws()
      ;
```

isNotDate().

```js
test
        .exception(function(){
          throw new Error('Whoops !');
        })
        .isNotDate()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw new Date();
            })
            .isNotDate()
          ;
        })
        .throws()
      ;
```

isArguments().

```js
var error = function(){
        return arguments;
      };
      test
        .exception(function(){
          throw error(1, 2, 3);
        })
        .isArguments()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw new Error('Whoops !');
            })
            .isArguments()
          ;
        })
        .throws()
      ;
```

isNotArguments().

```js
var error = function(){
        return arguments;
      };
      test
        .exception(function(){
          throw new Error('Whoops !');
        })
        .isNotArguments()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw error(1, 2, 3);
            })
            .isNotArguments()
          ;
        })
        .throws()
      ;
```

isEmpty().

```js
test
        .exception(function(){
          throw '';
        })
        .isEmpty()
        .exception(function(){
          throw [];
        })
        .isEmpty()
        .exception(function(){
          throw {};
        })
        .isEmpty()
        // Indeed an instance of `Error` has no enumerable properties.
        .exception(function(){
          throw new Error('Whoops !');
        })
        .isEmpty()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw 'Whoops !';
            })
            .isEmpty()
          ;
        })
        .throws()
      ;
```

isNotEmpty().

```js
test
        .exception(function(){
          throw 'Whoops !';
        })
        .isNotEmpty()
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw '';
                })
                .isNotEmpty()
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw [];
                })
                .isNotEmpty()
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw {};
                })
                .isNotEmpty()
              ;
            })
            .throws()
            .value(function(){
              // Indeed an instance of `Error` has no enumerable properties.
              test
                .exception(function(){
                  throw new Error('Whoops !');
                })
                .isNotEmpty()
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

isError().

```js
// isError() assertion is an alias of isInstanceOf(Error)
      var
        // create an indicator for monitoring the example and the test
        indicator = test.createCollection(),
        trigger = function(){
          indicator.set('error constructor called', true);
          throw new Error("I'm a ninja !");
        },
        resetIndicator = function(){
          // empty
          indicator.setAll({});
        }
      ;
      test
        // Assert that thrown with the Error class
        .exception(trigger)
          .isInstanceOf(Error)
        // just for the example and for the test
        .bool(indicator.get('error constructor called')).isTrue()
        // or shortcut
        .given(resetIndicator())
          // Assert that thrown with the Error class
          .exception(trigger)
            .isError()
          // just for the example and for the test
          .bool(indicator.get('error constructor called')).isTrue()
      ;
```

hasLength(expected).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .hasLength(2)
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw {message: 'error', code: 42};
            })
            .hasLength(1)
          ;
        })
        .throws()
      ;
```

hasNotLength(expected).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .hasNotLength(1)
        .hasNotLength(3)
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw {message: 'error', code: 42};
            })
            .hasNotLength(2)
          ;
        })
        .throws()
      ;
```

isEnumerable(property).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .isEnumerable('message')
        .isEnumerable('code')
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw new Error('Whoops !');
            })
            .isEnumerable('message')
          ;
        })
        .throws()
      ;
```

isNotEnumerable(property).

```js
test
        .exception(function(){
          throw new Error('Whoops !');
        })
        .isNotEnumerable('message')
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw {message: 'error', code: 42};
            })
            .isNotEnumerable('message')
          ;
        })
        .throws()
      ;
```

isFrozen().

```js
var
        error = {message: 'error', code: 42},
        frozenError = {message: 'error', code: 42}
      ;
      Object.freeze(frozenError);
      test
        .exception(function(){
          throw frozenError;
        })
        .isFrozen()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw error;
            })
            .isFrozen()
          ;
        })
        .throws()
      ;
```

isNotFrozen().

```js
var
        error = {message: 'error', code: 42},
        frozenError = {message: 'error', code: 42}
      ;
      Object.freeze(frozenError);
      test
        .exception(function(){
          throw error;
        })
        .isNotFrozen()
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw frozenError;
            })
            .isNotFrozen()
          ;
        })
        .throws()
      ;
```

isInstanceOf(expected).

```js
test
        .exception(function(){
          throw new TypeError('Whoops !');
        })
        .isInstanceOf(TypeError)
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw new Error('Bad type');
            })
            .isInstanceOf(TypeError)
          ;
        })
        .throws()
      ;
```

isNotInstanceOf(expected).

```js
test
        .exception(function(){
          throw new Error('Whoops !');
        })
        .isNotInstanceOf(TypeError)
        // Test failure
        .value(function(){
          test
            .exception(function(){
              throw new TypeError('Bad type');
            })
            .isNotInstanceOf(TypeError)
          ;
        })
        .throws()
      ;
```

hasProperty(property [, value]).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .hasProperty('message')
        .hasProperty('code', 42)
        .exception(function(){
          throw new Error('Whoops !');
        })
        .hasProperty('message')
        .hasProperty('message', 'Whoops !')
        .hasProperty('constructor')
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasProperty('foo')
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasProperty('code', 1)
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasNotProperty(property [, value]).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .hasNotProperty('foo')
        .hasNotProperty('code', 1)
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasNotProperty('message')
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasNotProperty('code', 42)
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw new Error('Whoops !');
                })
                .hasNotProperty('constructor')
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasOwnProperty(property [, value]).

```js
test
        .exception(function(){
          throw new Error('Whoops !');
        })
        .hasOwnProperty('message')
        .hasOwnProperty('message', 'Whoops !')
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw new Error('Whoops !');
                })
                .hasOwnProperty('foo')
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw new Error('Whoops !');
                })
                .hasOwnProperty('message', 'Grrrr !')
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw new Error('Whoops !');
                })
                .hasOwnProperty('constructor')
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasNotOwnProperty(property [, value]).

```js
test
        .exception(function(){
          throw new Error('Whoops !');
        })
        .hasNotOwnProperty('foo')
        .hasNotOwnProperty('message', 'Grrrr !')
        .hasNotOwnProperty('constructor')
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw new Error('Whoops !');
                })
                .hasNotOwnProperty('message')
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw new Error('Whoops !');
                })
                .hasNotOwnProperty('message', 'Whoops !')
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasProperties(properties).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .hasProperties(['message', 'code'])
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasProperties(['message', 'code', 'foo'])
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasProperties(['message'])
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasNotProperties(properties).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .hasNotProperties(['foo', 'bar'])
        .hasNotProperties(['foo', 'code', 'bar'])
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasNotProperties(['message', 'code'])
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasOwnProperties(properties).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .hasOwnProperties(['message', 'code'])
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasOwnProperties(['message', 'code', 'foo'])
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasOwnProperties(['message'])
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasKey(key [, value]).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .hasKey('message')
        .hasKey('code', 42)
        .exception(function(){
          throw new Error('Whoops !');
        })
        .hasKey('message')
        .hasKey('message', 'Whoops !')
        .hasKey('constructor')
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasKey('foo')
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasKey('code', 1)
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

notHasKey(key [, value]).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .notHasKey('foo')
        .notHasKey('code', 1)
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .notHasKey('message')
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .notHasKey('code', 42)
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw new Error('Whoops !');
                })
                .notHasKey('constructor')
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasKeys(keys).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .hasKeys(['message', 'code'])
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasKeys(['message', 'code', 'foo'])
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasKeys(['message'])
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

notHasKeys(keys).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .notHasKeys(['foo', 'bar'])
        .notHasKeys(['foo', 'code', 'bar'])
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .notHasKeys(['message', 'code'])
              ;
            })
            .throws()
          ;
        }) // end: Test failure
      ;
```

hasValue(expected).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .hasValue('error')
        .hasValue(42)
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasValue('err')
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasValue(2)
              ;
            })
            .throws()
        })
      ;
```

notHasValue(expected).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .notHasValue('err')
        .notHasValue(2)
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .notHasValue('error')
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .notHasValue(42)
              ;
            })
            .throws()
        })
      ;
```

hasValues(expected).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .hasValues(['error'])
        .hasValues(['error', 42])
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasValues(['foo'])
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .hasValues(['error', 42, 'foo'])
              ;
            })
            .throws()
        })
      ;
```

notHasValues(expected).

```js
test
        .exception(function(){
          throw {message: 'error', code: 42};
        })
        .notHasValues(['code'])
        .notHasValues(['message', 'code', 'foo'])
        .case('Test failure', function(){
          test
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .notHasValues(['error'])
              ;
            })
            .throws()
            .value(function(){
              test
                .exception(function(){
                  throw {message: 'error', code: 42};
                })
                .notHasValues(['foo', 'error'])
              ;
            })
            .throws()
        })
      ;
```

contains(expected [, ...]).

```js
test
        .exception(function(){
          throw new Error('Whoops');
        })
        .contains({message: 'Whoops'})
        // Test failure
        .value(function(){
            test
              .exception(function(){
                throw new Error('Whoops');
              })
              .contains({message: 'foo'})
            ;
          })
          .throws()
      ;
```

notContains(expected [, ...]).

```js
test
        .exception(function(){
          throw new Error('Whoops');
        })
        .notContains({message: 'foo'})
        // Test failure
        .value(function(){
            test
              .exception(function(){
                throw new Error('Whoops');
              })
              .notContains({message: 'Whoops'})
            ;
          })
          .throws()
      ;
```

startsWith(str).

```js
test
        .exception(function(){
          throw 'An error occured';
        })
        .startsWith('An error')
        // Test failure
        .value(function(){
            test
              .exception(function(){
                throw 'An error occured';
              })
              .startsWith('error')
            ;
          })
          .throws()
      ;
```

notStartsWith(str).

```js
test
        .exception(function(){
          throw 'An error occured';
        })
        .notStartsWith('error')
        // Test failure
        .value(function(){
            test
              .exception(function(){
                throw 'An error occured';
              })
              .notStartsWith('An error')
            ;
          })
          .throws()
      ;
```

endsWith(str).

```js
test
        .exception(function(){
          throw 'An error occured';
        })
        .endsWith('occured')
        // Test failure
        .value(function(){
            test
              .exception(function(){
                throw 'An error occured';
              })
              .endsWith('error')
            ;
          })
          .throws()
      ;
```

notEndsWith(str).

```js
test
        .exception(function(){
          throw 'An error occured';
        })
        .notEndsWith('error')
        // Test failure
        .value(function(){
            test
              .exception(function(){
                throw 'An error occured';
              })
              .notEndsWith('occured')
            ;
          })
          .throws()
      ;
```

hasMessage(expected).

```js
var
        // create an indicator for monitoring the example and the test
        indicator = test.createCollection(),
        trigger = function(){
          indicator.set('error constructor called', true);
          throw new Error("I'm a ninja !");
        },
        resetIndicator = function(){
          // empty
          indicator.setAll({});
        },
        exception
      ;
      test
        .exception(trigger)
          .hasMessage("I'm a ninja !")
        // just for the example and for the test
        .bool(indicator.get('error constructor called')).isTrue()
        // reset indicator
        // 'then' does nothing, is just pass-through method for a fluent chain.
        .then(resetIndicator())
          .exception(trigger)
            .hasMessage(/ninja/)
          // just for the example and for the test
          .bool(indicator.get('error constructor called')).isTrue()
        // reset indicator
        .then(resetIndicator())
        // out of trigger block,
        // because test.exception(trigger) throws the trigger in the block
        // also you can use test.value().throws() (see after)
        .case(exception = test.exception(trigger))
          .exception(function(){
            indicator.set('ninjaa is not in error message', true);
            // fails because ninjaa is not in error message
            exception.hasMessage(/ninjaa/);
          })
          // just for the example and for the test
          .bool(indicator.get('error constructor called')).isTrue()
          .bool(indicator.get('ninjaa is not in error message')).isTrue()
        // the above example is equal to
        .then(resetIndicator())
          .value(function(){
            indicator.set('ninjaa is not in error message', true);
            // fails because ninjaa is not in error message
            test.exception(trigger).hasMessage(/ninjaa/);
          })
          .throws()
          // just for the example and for the test
          .bool(indicator.get('error constructor called')).isTrue()
          .bool(indicator.get('ninjaa is not in error message')).isTrue()
      ;
```

<a name="asserter-function"></a>
# Asserter function()
<a name="asserter-function-function-behavior"></a>
## function() behavior
Does not contains assertions from the assertions containers.

```js
var fn = function(){};
        test
          .value(test.function(fn).hasHeader)
            .isUndefined()
          .value(test.function(fn).isAfter)
            .isUndefined()
          .value(test.function(fn).hasMessage)
            .isUndefined()
        ;
```

Assert that the tested value is a `function`.

```js
var fn = function(){};
      test
        .function(fn)
        .function(Date)
        .case('Test failure', function(){
          test
            .exception(function(){
              test.function(fn());
            })
            .exception(function(){
              test.function({});
            })
            .exception(function(){
              test.function([]);
            })
            .exception(function(){
              test.function(new Date());
            })
            .exception(function(){
              test.function('foobar');
            })
            .exception(function(){
              test.function(1);
            })
            .exception(function(){
              test.function(true);
            })
            .exception(function(){
              test.function(false);
            })
            .exception(function(){
              test.function(null);
            })
            .exception(function(){
              test.function(undefined);
            })
            .exception(function(){
              test.function();
            })
          ;
        })
      ;
```

<a name="asserter-function-assertions-of-function"></a>
## Assertions of function()
is(expected).

```js
var fn = function(){};
      var ref = fn;
      test
        .function(ref)
          .is(fn)
        .exception(function(){
          test.function(fn).is(function(){});
        })
      ;
```

isNot(expected).

```js
var fn = function(){};
      var otherFunction = function(){};
      test
        .function(fn)
          .isNot(otherFunction)
          .isNot(function(){})
        .exception(function(){
          test.function(fn).isNot(fn);
        })
      ;
```

isIdenticalTo(expected).

```js
var fn = function(){};
      var ref = fn;
      test
        .function(ref)
          .isIdenticalTo(fn)
        .exception(function(){
          test.function(fn).isIdenticalTo(function(){});
        })
      ;
```

isNotIdenticalTo(expected).

```js
var fn = function(){};
      var otherFunction = function(){};
      test
        .function(fn)
          .isNotIdenticalTo(otherFunction)
          .isNotIdenticalTo(function(){})
        .exception(function(){
          test.function(fn).isNotIdenticalTo(fn);
        })
      ;
```

isEqualTo(expected).

```js
var fn = function(){};
      var ref = fn;
      test
        .function(ref)
          .isEqualTo(fn)
        .exception(function(){
          test.function(fn).isEqualTo(function(){});
        })
      ;
```

isNotEqualTo(expected).

```js
var fn = function(){};
      var otherFunction = function(){};
      test
        .function(fn)
          .isNotEqualTo(otherFunction)
          .isNotEqualTo(function(){})
        .exception(function(){
          test.function(fn).isNotEqualTo(fn);
        })
      ;
```

match(expected).

```js
var fn = function(){
        return 'hello';
      };
      function myFunction(){
      };
      test
        .function(fn)
          .match('function')
          .match('func')
          .match(function(it){
            return it() === 'hello';
          })
        .function(Date)
          .match('Date')
        .function(myFunction)
          .match('myFunction')
          .match(/my/)
          .match(/[a-z]/i)
        .case('Test failure', function(){
          test
            .exception(function(){
              test.function(fn).match(function(it){
                return it() === 'hey';
              });
            })
            .exception(function(){
              test.function(fn).match('someFunction');
            })
            .exception(function(){
              test.function(myFunction).match(/someFunction/);
            })
          ;
        })
      ;
```

notMatch(expected).

```js
var fn = function(){
        return 'hello';
      };
      function myFunction(){
      };
      test
        .function(fn)
          .notMatch('foo')
          .notMatch(/[A-Z]/)
          .notMatch(function(it){
            return it() === 'hey';
          })
        .function(Date)
          .notMatch('foo')
        .function(myFunction)
          .notMatch('someFunction')
          .notMatch(/some/)
        .case('Test failure', function(){
          test
            .exception(function(){
              test.function(fn).notMatch(function(it){
                return it() === 'hello';
              });
            })
            .exception(function(){
              test.function(fn).notMatch('function');
            })
            .exception(function(){
              test.function(myFunction).notMatch(/myFunction/);
            })
            .exception(function(){
              test.function(myFunction).notMatch(/my/);
            })
            .exception(function(){
              test.function(myFunction).notMatch('myFunction');
            })
          ;
        })
      ;
```

isValid(expected).

```js
var fn = function(){
        return 'hello';
      };
      function myFunction(){
      };
      test
        .function(fn)
          .isValid('function')
          .isValid('func')
          .isValid(function(it){
            return it() === 'hello';
          })
        .function(Date)
          .isValid('Date')
        .function(myFunction)
          .isValid('myFunction')
          .isValid(/my/)
          .isValid(/[a-z]/i)
        .case('Test failure', function(){
          test
            .exception(function(){
              test.function(fn).isValid(function(it){
                return it() === 'hey';
              });
            })
            .exception(function(){
              test.function(fn).isValid('someFunction');
            })
            .exception(function(){
              test.function(myFunction).isValid(/someFunction/);
            })
          ;
        })
      ;
```

isNotValid(expected).

```js
var fn = function(){
        return 'hello';
      };
      function myFunction(){
      };
      test
        .function(fn)
          .isNotValid('foo')
          .isNotValid(/[A-Z]/)
          .isNotValid(function(it){
            return it() === 'hey';
          })
        .function(Date)
          .isNotValid('foo')
        .function(myFunction)
          .isNotValid('someFunction')
          .isNotValid(/some/)
        .case('Test failure', function(){
          test
            .exception(function(){
              test.function(fn).isNotValid(function(it){
                return it() === 'hello';
              });
            })
            .exception(function(){
              test.function(fn).isNotValid('function');
            })
            .exception(function(){
              test.function(myFunction).isNotValid(/myFunction/);
            })
            .exception(function(){
              test.function(myFunction).isNotValid(/my/);
            })
            .exception(function(){
              test.function(myFunction).isNotValid('myFunction');
            })
          ;
        })
      ;
```

throws([constructor|expected], [expected]).

```js
var fn = function(){};
      var trigger = function(){
        throw new Error('Whoops!');
      };
      test
        .function(trigger)
          .throws()
          .throws('Whoops!')
          .throws(/whoops/i)
          .throws(Error)
          .throws(Error, /whoops/i)
        .case('Test failure', function(){
          test
            .value(function(){
              test.function(fn).throws();
            })
            .throws()
            .value(function(){
              test.function(trigger).throws(TypeError);
            })
            .throws()
            .value(function(){
              test.function(trigger).throws('gloops');
            })
            .throws()
            .value(function(){
              test.function(trigger).throws(/gloops/);
            })
            .throws()
            .value(function(){
              test.function(trigger).throws(TypeError, 'gloops');
            })
            .throws()
            .value(function(){
              test.function(trigger).throws(Error, 'whoops');
            })
            .throws()
          ;
        })
      ;
```

isError().

```js
var trigger = function(){
        throw new Error('Whoops!');
      };
      test
        .function(trigger)
          .isError()
        .case('Test failure', function(){
          test
            .value(function(){
              test.function(function(){}).isError();
            })
            .throws()
            .value(function(){
              test
                .function(function(){
                  throw 'error';
                })
                .isError()
              ;
            })
            .throws()
          ;
        })
      ;
```

hasName(expected).

```js
var fn = function(){};
      function myFunction(){
      };
      test
        .function(Date)
          .hasName('Date')
        .function(myFunction)
          .hasName('myFunction')
        .case('Test failure', function(){
          test
            .exception(function(){
              test.function(new Date()).hasName('RegExp');
            })
            .exception(function(){
              test.function(myFunction).hasName('function');
            })
          ;
        })
      ;
```

<a name="asserter-number"></a>
# Asserter number()
<a name="asserter-number-number-behavior"></a>
## number() behavior
Does not contains assertions from the assertions containers.

```js
test
        .value(test.number(1).hasHeader)
          .isUndefined()
        .value(test.number(1).hasProperty)
          .isUndefined()
        .value(test.number(1).hasMessage)
          .isUndefined()
      ;
```

Assert that the tested value is a `number`.

```js
test
        .number(2)
        .number(99.98)
        .number(NaN)
        .case('Test failure', function(){
          test
            .exception(function(){
              test.number('0');
            })
            .exception(function(){
              test.number('1');
            })
            .exception(function(){
              test.number(/0/);
            })
            .exception(function(){
              test.number(/1/);
            })
            .exception(function(){
              test.number(true);
            })
            .exception(function(){
              test.number(false);
            })
            .exception(function(){
              test.number(null);
            })
            .exception(function(){
              test.number(undefined);
            })
            .exception(function(){
              test.number({});
            })
            .exception(function(){
              test.number([]);
            })
          ;
        })
      ;
```

<a name="asserter-number-assertions-of-number"></a>
## Assertions of number()
is(expected).

```js
test
        .number(2)
          .is(2)
        .case('Test failure', function(){
          test
            .exception(function(){
              test.number(2).is(-2)
            })
            .exception(function(){
              test.number(2).is(2.02)
            })
          ;
        })
      ;
```

isNot(expected).

```js
test
        .number(2)
          .isNot(3)
          .isNot('2')
          .isNot(2.1)
          .isNot(0.2)
          .isNot(0.02)
          .isNot(-2)
        .exception(function(){
          test.number(2).isNot(2);
        })
      ;
```

isIdenticalTo(expected).

```js
test
        .number(1)
          .isIdenticalTo(1)
         .case('Test failure', function(){
          test
            .exception(function(){
              test.number(2).isIdenticalTo(-2)
            })
            .exception(function(){
              test.number(2).isIdenticalTo(2.02)
            })
          ;
        })
      ;
```

isNotIdenticalTo(expected).

```js
test
        .number(2)
          .isNotIdenticalTo(3)
          .isNotIdenticalTo('2')
          .isNotIdenticalTo(2.1)
          .isNotIdenticalTo(0.2)
          .isNotIdenticalTo(-2)
        .exception(function(){
          test.number(2).isNotIdenticalTo(2);
        })
      ;
```

isEqualTo(expected).

```js
test
        .number(1)
          .isEqualTo(1)
          .isEqualTo('1')
        .case('Test failure', function(){
          test
            .exception(function(){
              test.number(2).isEqualTo(-2)
            })
            .exception(function(){
              test.number(2).isEqualTo(2.02)
            })
          ;
        })
     ;
```

isNotEqualTo(expected).

```js
test
        .number(2)
          .isNotEqualTo(3)
          .isNotEqualTo(-2)
          .isNotEqualTo(2.1)
          .isNotEqualTo('2.1')
        .exception(function(){
          test.number(2).isNotEqualTo(2);
        })
      ;
```

match(expected).

```js
test
        // Assert with a RegExp
        .number(2014).match(/20+[1-4]/)
        // Assert with a number converted to RegExp
        .number(2014).match(201)
        // Assert with a function
        .number(2014).match(function(it){
          return it === 2014;
        })
        .exception(function(){
          test.number(2).match(/3/);
        })
      ;
```

notMatch(expected).

```js
test
        // Assert with a RegExp
        .number(2014)
          .notMatch(/20+[5-6]/)
          .notMatch(/[a-z]/)
        // Assert with a number converted to RegExp
        .number(10)
          .notMatch(8)
        // Assert with a function
        .number(10)
          .notMatch(function(it){
            return it === 42;
          })
        .case('Test failure', function(){
          test
            .exception(function(){
              test.number(2).notMatch(/2/);
            })
            .exception(function(){
              test.number(2014).notMatch(function(it){
                return it === 2014;
              });
            })
          ;
        })
      ;
```

isValid(expected).

```js
test
        // Assert with a RegExp
        .number(2014).isValid(/20+[1-4]/)
        // Assert with a number converted to RegExp
        .number(2014).isValid(201)
        // Assert with a function
        .number(2014).isValid(function(it){
          return it === 2014;
        })
        .exception(function(){
          test.number(2).isValid(/3/);
        })
      ;
```

isNotValid(expected).

```js
test
        // Assert with a RegExp
        .number(2014)
          .isNotValid(/20+[5-6]/)
          .isNotValid(/[a-z]/)
        // Assert with a number converted to RegExp
        .number(10)
          .isNotValid(8)
        // Assert with a function
        .number(10)
          .isNotValid(function(it){
            return it === 42;
          })
        .case('Test failure', function(){
          test
            .exception(function(){
              test.number(2).isNotValid(/2/);
            })
            .exception(function(){
              test.number(2014).isNotValid(function(it){
                return it === 2014;
              });
            })
          ;
        })
      ;
```

matchEach(expected).

```js
test
        .number(2014)
          .matchEach([2, 4, 1, 0])
          .matchEach([2014, function(it){
            return it === 2014;
          }])
        .case('Test failure', function(){
          test
            .exception(function(){
              test.number(2014).matchEach([2014, function(it){
                return it === 2041;
              }]);
            })
            .exception(function(){
              test.number(2).matchEach([2, 3]);
            })
            .exception(function(){
              test.number(2014).matchEach([2041, function(it){
                return it === 2014;
              }]);
            })
          ;
        })
      ;
```

notMatchEach(expected).

```js
test
        .number(2014)
          .notMatchEach([3, 200])
          .notMatchEach([2012, function(it){
            return it !== 2014;
          }])
        .case('Test failure', function(){
          test
            .exception(function(){
              test.number(2).notMatchEach([2, 3]);
            })
            .exception(function(){
              test.number(2014).notMatchEach([2041, function(it){
                return it === 2014;
              }]);
            })
            .exception(function(){
              test.number(2014).notMatchEach([2012, function(it){
                return it === 2014;
              }]);
            })
          ;
        })
      ;
```

isBetween(begin, end).

```js
test
        .number(2)
          .isBetween(2, 4)
        .number(3)
          .isBetween(2, 4)
        .number(4)
          .isBetween(2, 4)
        .number(4)
          .isBetween(3.99, 4.01)
        .number(1)
          .isBetween(-1.00000001, 1.0000000001)
        .case('Test failure', function(){
          test
            .exception(function(){
              test.number(2).isBetween(-3, -1);
            })
            .exception(function(){
              test.number(2).isBetween(3, 1);
            })
          ;
        })
      ;
```

isNotBetween(begin, end).

```js
test
        .number(1)
          .isNotBetween(2, 4)
        .number(5)
          .isNotBetween(2, 4)
        .exception(function(){
          test.number(2).isNotBetween(1, 3);
        })
      ;
```

isBefore(expected).

```js
test
        .number(1)
          .isBefore(2)
          .isBefore(1.01)
        .exception(function(){
          test.number(1).isBefore(0);
        })
      ;
```

isAfter(expected).

```js
test
        .number(2)
          .isAfter(1)
          .isAfter(1.99)
          .isAfter(-3)
        .exception(function(){
          test.number(-1).isAfter(0);
        })
      ;
```

isLessThan(expected).

```js
test
        .number(1)
          .isLessThan(2)
          .isLessThan(1.01)
        .exception(function(){
          test.number(1).isLessThan(0);
        })
      ;
```

isGreaterThan(expected).

```js
test
        .number(2)
          .isGreaterThan(1)
          .isGreaterThan(1.99)
          .isGreaterThan(-3)
        .exception(function(){
          test.number(-1).isGreaterThan(0);
        })
      ;
```

isApprox(num, delta).

```js
test
        .number(99.98)
          .isApprox(100, 0.02)
        .exception(function(){
          test.number(99.98).isApprox(100, 0.01);
        })
      ;
```

isInfinite().

```js
test
        .number(1/0)
          .isInfinite()
        .exception(function(){
          test.number(1.333333333333333333333).isInfinite();
        })
      ;
```

isNotInfinite().

```js
test
        .number(1.333333333333333333333333)
          .isNotInfinite()
        .exception(function(){
          test.number(1/0).isNotInfinite();
        })
      ;
```

isNaN().

```js
test
  .number(NaN)
    .isNaN()
  .number(new Number(NaN))
    .isNaN()
  .number(0/0)
    .isNaN()
  .number(parseInt('a', 10))
    .isNaN()
  .exception(function(){
    test.number(1).isNaN();
  })
  .exception(function(){
    test.number(new Number(1)).isNaN();
  })
  .exception(function(){
    test.number(0).isNaN();
  })
;
```

isNotNaN().

```js
test
  .number(1)
    .isNotNaN()
  .number(new Number(1))
    .isNotNaN()
  .number(0)
    .isNotNaN()
  .exception(function(){
    test.number(NaN).isNotNaN();
  })
  .exception(function(){
    test.number(new Number(NaN)).isNotNaN();
  })
  .exception(function(){
    test.number(0/0).isNotNaN();
  })
  .exception(function(){
    test.number(parseInt('a', 10)).isNotNaN();
  })
;
```

<a name="asserter-object"></a>
# Asserter object()
<a name="asserter-object-object-behavior"></a>
## object() behavior
Does not contains assertions from the assertions containers.

```js
test
        .value(test.object({}).hasHeader)
          .isUndefined()
        .value(test.object({}).isError)
          .isUndefined()
        .value(test.object({}).hasMessage)
          .isUndefined()
        .value(test.object({}).isInfinite)
          .isUndefined()
      ;
```

Assert that the tested value is an `object`.

```js
var Foo = function Foo(){};
      test
        .object({})
        .object([])
        .object(new Date())
        .object(new RegExp())
        .object(new Foo())
        .case('Test failure', function(){
          test
            .exception(function(){
              test.object('Foo');
            })
            .exception(function(){
              test.object(Foo);
            })
            .exception(function(){
              test.object(1);
            })
            .exception(function(){
              test.object(undefined);
            })
            .exception(function(){
              test.object(true);
            })
            .exception(function(){
              test.object(false);
            })
            .exception(function(){
              test.object(null);
            })
            .exception(function(){
              test.object(function(){});
            })
          ;
        })
      ;
```

<a name="asserter-object-assertions-of-object"></a>
## Assertions of object()
is(expected).

```js
test
        .object({fluent: 'is awesome', deep: [0, 1]})
          .is({fluent: 'is awesome', deep: [0, 1]})
        .exception(function(){
          test.object({fluent: 'is awesome', deep: [0, 1]})
            .is({fluent: 'is awesome', deep: [0, 2]});
        })
      ;
```

isNot(expected).

```js
test
        .object({fluent: 'is awesome', deep: [0, 1]})
          .isNot({fluent: 'is awesome', deep: [0, '1']})
        .exception(function(){
          test.object({fluent: 'is awesome', deep: [0, 1]})
            .isNot({fluent: 'is awesome', deep: [0, 1]});
        })
      ;
```

isIdenticalTo(expected).

```js
var
  obj = {},
  obj2 = obj
;
test
  .object(obj)
    .isIdenticalTo(obj2)
  .exception(function(){
    test.object(obj).isIdenticalTo({});
  })
;
```

isNotIdenticalTo(expected).

```js
var
        obj = {},
        obj2 = obj
      ;
      test
        .object(obj)
          .isNotIdenticalTo({})
        .exception(function(){
          test.object(obj).isNotIdenticalTo(obj2);
        })
      ;
```

isEqualTo(expected).

```js
var
  obj = {},
  obj2 = obj
;
test
  .object(obj)
    .isEqualTo(obj2)
  .exception(function(){
    test.object(obj).isEqualTo({});
  })
;
```

isNotEqualTo(expected).

```js
var
  obj = {foo: 'bar'},
  obj2 = obj
;
test
  .object(obj)
    .isNotEqualTo({foo: 'bar', baz: 'bar'})
  .exception(function(){
    test.object(obj).isNotEqualTo(obj2);
  })
;
```

match(expected).

```js
test
        .object({hello: 'world'})
          .match(function(obj){
            return obj.hello == 'world';
          })
        .exception(function(){
          test.object({hello: 'world'})
            .match(function(obj){
              return obj.hello == 'foo';
            });
        })
      ;
```

notMatch(expected).

```js
test
        .object({hello: 'world'})
          .notMatch(function(obj){
            return obj.hello == 'E.T';
          })
        .exception(function(){
          test.object({hello: 'world'})
            .notMatch(function(obj){
              return obj.hello == 'world';
            })
        })
      ;
```

isValid(expected).

```js
test
        .object({hello: 'world'})
          .isValid(function(obj){
            return obj.hello == 'world';
          })
        .exception(function(){
          test.object({hello: 'world'})
            .isValid(function(obj){
              return obj.hello == 'foo';
            });
        })
      ;
```

isNotValid(expected).

```js
test
        .object({hello: 'world'})
          .isNotValid(function(obj){
            return obj.hello == 'E.T';
          })
        .exception(function(){
          test.object({hello: 'world'})
            .isNotValid(function(obj){
              return obj.hello == 'world';
            })
        })
      ;
```

matchEach(expected).

```js
test
        .object({foo: 'bar', hey: 'you', joker:1})
          .matchEach(function(it, key) {
            if(key == 'joker'){
              return (typeof it == 'number');
            }
            return (typeof it == 'string');
          })
        .exception(function(){
          // error if one or several does not match
          test.object({foo: 'bar', hey: 'you', joker:1})
            .matchEach(function(it, key) {
              return (typeof it == 'string');
            })
        })
      ;
```

notMatchEach(expected).

```js
test
        .object({foo: 'bar', hey: 'you', joker:1})
          .notMatchEach(function(it, key) {
            if(key == 'other'){
              return true;
            }
          })
        .exception(function(){
          // error if one or several does not match
          test.object({foo: 'bar', hey: 'you', joker:1})
            .notMatchEach(function(it, key) {
              if(key == 'foo'){
                return true;
              }
            })
        })
        .exception(function(){
          // error if one or several does not match
          test.object({foo: 'bar', hey: 'you', joker:1})
            .notMatchEach(function(it, key) {
              return (typeof it == 'string');
            })
        })
      ;
```

isArray().

```js
test
  .object([])
    .isArray()
  .exception(function(){
    test.object({}).isArray();
  })
  .exception(function(){
    test.object(new Date()).isArray();
  })
;
```

isRegExp().

```js
test
  .object(/[0-9]+/)
    .isRegExp()
  .exception(function(){
    test.object({}).isRegExp();
  })
  .exception(function(){
    test.object(new Date()).isRegExp();
  })
;
```

isNotRegExp().

```js
test
  .object(new Date())
    .isNotRegExp()
  .exception(function(){
     test.object(/[0-9]+/).isNotRegExp();
  })
;
```

isDate().

```js
test
  .object(new Date())
    .isDate()
  .exception(function(){
    test.object({}).isDate();
  })
  .exception(function(){
     test.object(/[0-9]+/).isDate();
  })
;
```

isNotDate().

```js
test
  .object(/[0-9]+/)
    .isNotDate()
  .exception(function(){
    test.object(new Date()).isNotDate();
  })
;
```

isArguments().

```js
var fn = function(){
        var args = arguments;
        test.object(arguments).isArguments();
        test.object(args).isArguments();
      };
      fn(1, 2, 3);
      test.exception(function(){
        test.object({0: 'a'}).isArguments();
      });
```

isNotArguments().

```js
var fn = function(){
        test
          .object(arguments)
            .isArguments()
          .object([1, 2, 3])
            .isNotArguments()
          .object({0:1, 1:2, 2:3})
            .isNotArguments()
        ;
      };
      fn(1, 2, 3);
      test.exception(function(){
        test.object(arguments).isNotArguments();
      });
```

isEmpty().

```js
test
  .object({})
    .isEmpty()
  .exception(function(){
    test.object({0: 'a'}).isEmpty();
  })
;
```

isNotEmpty().

```js
test
  .object({hello: 'Nico'})
    .isNotEmpty()
  .exception(function(){
    test.object({}).isNotEmpty();
  })
;
```

hasLength(expected).

```js
test
        .object({foo: 'bar', other: 'baz'})
          .hasLength(2)
        .exception(function(){
          test.object({foo: 'bar', other: 'baz'})
            .hasLength(3);
        })
      ;
```

hasNotLength(expected).

```js
test
        .object({foo: 'bar', other: 'baz'})
          .hasNotLength(4)
        .exception(function(){
          test.object({foo: 'bar', other: 'baz'})
            .hasNotLength(2);
        })
      ;
```

isEnumerable(property).

```js
test
        .object({prop: 'foobar'})
          .isEnumerable('prop')
        .exception(function(){
          test.object({prop: 'foobar'})
            .isEnumerable('length');
        })
      ;
```

isNotEnumerable(property).

```js
test
        .object(Object.create({}, {prop: {enumerable: 0}}))
          .isNotEnumerable('prop')
        .exception(function(){
          test.object({prop: 'foobar'})
            .isNotEnumerable('prop');
        })
      ;
```

isFrozen().

```js
test
  .object(Object.freeze({}))
    .isFrozen()
  .exception(function(){
    test.object({})
      .isFrozen();
  })
;
```

isNotFrozen().

```js
test
  .object({})
    .isNotFrozen()
  .exception(function(){
    test.object(Object.freeze({}))
      .isNotFrozen();
  })
;
```

isInstanceOf(expected).

```js
test
  .object(new Date())
    .isInstanceOf(Date)
  .exception(function(){
    test.object(new Date())
      .isInstanceOf(Error);
  })
;
```

isNotInstanceOf(expected).

```js
test
  .object(new Date())
    .isNotInstanceOf(RegExp)
  .exception(function(){
    test.object(new Date())
      .isNotInstanceOf(Object);
  })
  .exception(function(){
    test.object(new Date())
      .isNotInstanceOf(Date);
  })
;
```

hasProperty(property [, value]).

```js
test
  .object({foo: 'bar'})
    .hasProperty('foo')
  .object({foo: 'bar'})
    .hasProperty('foo', 'bar')
  .exception(function(){
    test.object({foo: 'bar'})
      .hasProperty('bar');
  })
  .exception(function(){
    test.object({foo: 'bar'})
      .hasProperty('foo', 'ba');
  })
;
```

hasNotProperty(property [, value]).

```js
test
  .object({foo: 'bar'})
    .hasNotProperty('bar')
  .object({foo: 'bar'})
    .hasNotProperty('foo', 'baz')
  .exception(function(){
    test.object({foo: 'bar'})
      .hasNotProperty('foo');
  })
  .exception(function(){
    test.object({foo: 'bar'})
      .hasNotProperty('foo', 'bar');
  })
;
```

hasOwnProperty(property [, value]).

```js
test
  .object({foo: 'bar'})
    .hasOwnProperty('foo')
  .object({foo: 'bar'})
    .hasOwnProperty('foo', 'bar')
  .exception(function(){
    test.object({foo: 'bar'})
      .hasOwnProperty('bar');
  })
  .exception(function(){
    test.object({foo: 'bar'})
      .hasOwnProperty('foo', 'ba');
  })
;
```

hasNotOwnProperty(property [, value]).

```js
test
  .object({foo: 'bar'})
    .hasNotOwnProperty('bar')
  .object({foo: 'bar'})
    .hasNotOwnProperty('foo', 'baz')
  .exception(function(){
    test.object({foo: 'bar'})
      .hasNotOwnProperty('foo');
  })
  .exception(function(){
    test.object({foo: 'bar'})
      .hasNotOwnProperty('foo', 'bar');
  })
;
```

hasProperties(properties).

```js
test
  .object({foo: 'bar', bar: 'huhu', other: 'vroom'})
    .hasProperties(['other', 'bar', 'foo'])
  .exception(function(){
    test.object({foo: 'bar', bar: 'huhu', other: 'vroom'})
      .hasProperties(['other', 'bar']);
  })
;
```

hasNotProperties(properties).

```js
test
  .object({foo: 'bar', bar: 'huhu', other: 'vroom'})
    .hasNotProperties(['other', 'foo'])
  .exception(function(){
    test.object({foo: 'bar', bar: 'huhu', other: 'vroom'})
      .hasNotProperties(['bar', 'other', 'foo']);
  })
;
```

hasOwnProperties(properties).

```js
test
  .object({foo: 'bar', bar: 'huhu', other: 'vroom'})
    .hasOwnProperties(['other', 'bar', 'foo'])
  .exception(function(){
    test.object({foo: 'bar', bar: 'huhu', other: 'vroom'})
      .hasOwnProperties(['other', 'bar']);
  })
;
```

hasKey(key [, value]).

```js
test
  .object({foo: 'bar'})
    .hasKey('foo')
  .object({foo: 'bar'})
    .hasKey('foo', 'bar')
  .exception(function(){
    test.object({foo: 'bar'})
      .hasKey('bar');
  })
  .exception(function(){
    test.object({foo: 'bar'})
      .hasKey('foo', 'ba');
  })
;
```

notHasKey(key [, value]).

```js
test
  .object({foo: 'bar'})
    .notHasKey('bar')
  .object({foo: 'bar'})
    .notHasKey('foo', 'baz')
  .exception(function(){
    test.object({foo: 'bar'})
      .notHasKey('foo');
  })
  .exception(function(){
    test.object({foo: 'bar'})
      .notHasKey('foo', 'bar');
  })
;
```

hasKeys(keys).

```js
test
  .object({foo: 'bar', bar: 'huhu', other: 'vroom'})
    .hasKeys(['other', 'bar', 'foo'])
  .exception(function(){
    test.object({foo: 'bar', bar: 'huhu', other: 'vroom'})
      .hasKeys(['other', 'bar']);
  })
;
```

notHasKeys(keys).

```js
test
  .object({foo: 'bar', bar: 'huhu', other: 'vroom'})
    .notHasKeys(['other', 'foo'])
  .exception(function(){
    test.object({foo: 'bar', bar: 'huhu', other: 'vroom'})
      .notHasKeys(['bar', 'other', 'foo']);
  })
;
```

hasValue(expected).

```js
test
        .object({life: 42, love: 69})
          .hasValue(42)
        .exception(function(){
          test.object({life: 42, love: 69})
            .hasValue('42');
        })
      ;
```

notHasValue(expected).

```js
test
        .object({life: 42, love: 69})
          .notHasValue(4)
        .exception(function(){
          test.object({life: 42, love: 69})
            .notHasValue(42);
        })
      ;
```

hasValues(expected).

```js
test
        .object({life: 42, love: 69})
          .hasValues([42, 69])
        .exception(function(){
          test.object([1, 42, 3])
            .hasValues([42, 3.01]);
        })
      ;
```

notHasValues(expected).

```js
test
        .object({life: 42, love: 69})
          .notHasValues([43, 68])
        .exception(function(){
          test.object([1, 42, 3])
            .notHasValues([1, 42, 3]);
        })
      ;
```

contains(expected [, ...]).

```js
test
        .object({ a: { b: 10 }, b: { c: 10, d: 11, a: { b: 10, c: 11} }})
        .contains({ a: { b: 10 }, b: { c: 10, a: { c: 11 }}})
        .object({a: 'a', b: {c: 'c'}})
          .contains({b: {c: 'c'}})
          .contains({b: {c: 'c'}}, {a: 'a'})
        .exception(function(){
          test.object({foo: {a: 'a'}, bar: {b: 'b', c: 'c'}})
            .contains({foo: {a: 'a'}}, {bar: {b: 'c'}});
        })
      ;
```

notContains(expected [, ...]).

```js
test
        .object({a: 'a'}, {b: 'b', c: 'c'})
          .notContains({c: 'b'})
        .exception(function(){
          test.object({foo: {a: 'a'}, bar: {b: 'b', c: 'c'}})
            .notContains({foo: {a: 'a'}, bar: {c: 'c'}});
        })
      ;
```

hasName(expected).

```js
test
        .object(new Date(2010, 5, 28))
          .hasName('Date')
        .exception(function(){
          test.object(new Date(2010, 5, 28))
            .hasName('date');
        })
      ;
```

<a name="asserter-regexp"></a>
# Asserter regexp()
<a name="asserter-regexp-regexp-behavior"></a>
## regexp() behavior
Does not contains assertions from the assertions containers.

```js
test
        .value(test.regexp(new RegExp()).hasHeader)
          .isUndefined()
        .value(test.regexp(new RegExp()).hasMessage)
          .isUndefined()
        .value(test.regexp(new RegExp()).isInfinite)
          .isUndefined()
      ;
```

Assert that the tested value is an instance of `RegExp`.

```js
test
        .regexp(/foobar/)
        .regexp(new RegExp('foo', 'i'))
        .case('Test failure', function(){
          test
            .exception(function(){
              test.regexp({});
            })
            .exception(function(){
              test.regexp([]);
            })
            .exception(function(){
              test.regexp('');
            })
            .exception(function(){
              test.regexp();
            })
            .exception(function(){
              test.regexp(undefined);
            })
            .exception(function(){
              test.regexp(null);
            })
            .exception(function(){
              test.regexp(true);
            })
            .exception(function(){
              test.regexp(false);
            })
            .exception(function(){
              test.regexp(0);
            })
            .exception(function(){
              test.regexp(1);
            })
            .exception(function(){
              test.regexp('[a-z]');
            })
            .exception(function(){
              test.regexp('/foobar/');
            })
            .exception(function(){
              test.regexp(RegExp);
            })
            .exception(function(){
              test.regexp(new Date());
            })
          ;
        })
      ;
```

<a name="asserter-regexp-assertions-of-regexp"></a>
## Assertions of regexp()
is(expected).

```js
var regexp = new RegExp(/[a-z]/);
      test
        .regexp(regexp)
          .is(new RegExp(/[a-z]/))
        .exception(function(){
          test.regexp(regexp).is(/A-Z/);
        })
      ;
```

isNot(expected).

```js
var regexp = new RegExp(/[a-z]/);
      test
        .regexp(regexp)
          .isNot(new RegExp(/[A-Z]/))
        .exception(function(){
          test.regexp(regexp).isNot(/[a-z]/);
        })
      ;
```

isIdenticalTo(expected).

```js
var
        regexp = new RegExp(/[a-z]/),
        ref = regexp
      ;
      test
        .regexp(regexp)
          .isIdenticalTo(ref)
        .exception(function(){
          test.regexp(regexp).isIdenticalTo(/[a-z]/);
        })
      ;
```

isNotIdenticalTo(expected).

```js
var regexp = new RegExp(/[a-z]/);
      test
        .regexp(regexp)
          .isNotIdenticalTo(new RegExp(/[a-z]/))
        .exception(function(){
          test.regexp(regexp).isNotIdenticalTo(regexp);
        })
      ;
```

isEqualTo(expected).

```js
var
        regexp = new RegExp(/[a-z]/),
        ref = regexp
      ;
      test
        .regexp(regexp)
          .isEqualTo(ref)
        .exception(function(){
          test.regexp(regexp).isEqualTo(/[a-z]/);
        })
      ;
```

isNotEqualTo(expected).

```js
var regexp = new RegExp(/[a-z]/);
      test
        .regexp(regexp)
          .isNotEqualTo(new RegExp(/[a-z]/))
        .exception(function(){
          test.regexp(regexp).isNotEqualTo(regexp);
        })
      ;
```

match(expected).

```js
var
        regexp = new RegExp(/[a-z]/),
        ref = regexp
      ;
      test
        .regexp(regexp)
          .match(function(reg){
            return reg === ref;
          })
        .exception(function(){
          test.regexp(regexp).match(function(actual){
            return actual.source == '[A-Z]';
          });
        })
      ;
```

notMatch(expected).

```js
var
        regexp = new RegExp(/[a-z]/),
        ref = regexp
      ;
      test
        .regexp(regexp)
          .notMatch(function(actual){
            var reg = new RegExp(/[a-z]/);
            return reg === ref || reg === actual;
          })
        .exception(function(){
          test.regexp(regexp).notMatch(function(actual){
            return actual.source == '[a-z]';
          });
        })
      ;
```

isValid(expected).

```js
var
        regexp = new RegExp(/[a-z]/),
        ref = regexp
      ;
      test
        .regexp(regexp)
          .isValid(function(reg){
            return reg === ref;
          })
        .exception(function(){
          test.regexp(regexp).isValid(function(actual){
            return actual.source == '[A-Z]';
          });
        })
      ;
```

isNotValid(expected).

```js
var
        regexp = new RegExp(/[a-z]/),
        ref = regexp
      ;
      test
        .regexp(regexp)
          .isNotValid(function(actual){
            var reg = new RegExp(/[a-z]/);
            return reg === ref || reg === actual;
          })
        .exception(function(){
          test.regexp(regexp).isNotValid(function(actual){
            return actual.source == '[a-z]';
          });
        })
      ;
```

isEnumerable(property).

```js
var regexp = new RegExp(/[a-z]/);
      // define an enumerable property
      Object.defineProperty(regexp, 'myCustom', {
        enumerable: true, value: 'static'
      });
      test
        .regexp(regexp)
          .isEnumerable('myCustom')
        .exception(function(){
          test.regexp(regexp).isEnumerable('ignoreCase');
        })
      ;
```

isNotEnumerable(property).

```js
var regexp = new RegExp(/[a-z]/);
      // define an enumerable property
      Object.defineProperty(regexp, 'myCustom', {
        enumerable: true, value: 'static'
      });
      test
        .regexp(regexp)
          .isNotEnumerable('lastIndex')
          .isNotEnumerable('ignoreCase')
          .isNotEnumerable('multiline')
        .exception(function(){
          test.regexp(regexp).isNotEnumerable('myCustom');
        })
      ;
```

isFrozen.

```js
var regexp = new RegExp(/[a-z]/);
      Object.freeze(regexp);
      test
        .regexp(regexp)
          .isFrozen()
        .exception(function(){
          test.regexp(/[a-z]/).isFrozen();
        })
      ;
```

isNotFrozen.

```js
var
        regexp = new RegExp(/[a-z]/),
        regexpFrozen = new RegExp(/[a-z]/)
      ;
      Object.freeze(regexpFrozen);
      test
        .regexp(regexp)
          .isNotFrozen()
        .exception(function(){
          test.regexp(regexpFrozen).isNotFrozen();
        })
      ;
```

hasProperty(property [, value]).

```js
test
        .regexp(/[a-z]/)
          .hasProperty('lastIndex')
          .hasProperty('constructor')
        .exception(function(){
          test.regexp(/[a-z]/).hasProperty('foo');
        })
      ;
```

hasNotProperty(property [, value]).

```js
test
  .regexp(/[a-z]/)
    .hasNotProperty('foobar')
  .exception(function(){
    test.regexp(/[a-z]/).hasNotProperty('lastIndex');
  })
;
```

hasOwnProperty(property [, value]).

```js
test
  .regexp(/[a-z]/)
    .hasOwnProperty('lastIndex')
  .exception(function(){
    test.regexp(/[a-z]/).hasOwnProperty('constructor');
  })
;
```

hasNotOwnProperty(property [, value]).

```js
test
  .regexp(/[a-z]/)
    .hasNotOwnProperty('constructor')
  .exception(function(){
    test.regexp(/[a-z]/).hasNotOwnProperty('lastIndex');
  })
;
```

hasKey(key [,value ]).

```js
test
  .regexp(/[a-z]/)
    .hasKey('lastIndex')
    .hasKey('constructor')
  .exception(function(){
    test.regexp(/[a-z]/).hasKey('foo');
  })
;
```

notHasKey(key [,value]).

```js
test
  .regexp(/[a-z]/)
    .notHasKey('foobar')
  .exception(function(){
    test.regexp(/[a-z]/).notHasKey('lastIndex');
  })
;
```

<a name="asserter-string"></a>
# Asserter string()
<a name="asserter-string-string-behavior"></a>
## string() behavior
Does not contains assertions from the assertions containers.

```js
test
        .value(test.string('').hasHeader)
          .isUndefined()
        .value(test.string('').hasProperty)
          .isUndefined()
        .value(test.string('').hasMessage)
          .isUndefined()
        .value(test.string('').isInfinite)
          .isUndefined()
      ;
```

Assert that the tested value is a `string`.

```js
test
        .string('')
        .string('Hello')
        .case('Test failure', function(){
          test
            .exception(function(){
              test.string();
            })
            .exception(function(){
              test.string({});
            })
            .exception(function(){
              test.string([]);
            })
            .exception(function(){
              test.string(1);
            })
            .exception(function(){
              test.string(/foobar/);
            })
            .exception(function(){
              test.string(true);
            })
            .exception(function(){
              test.string(false);
            })
            .exception(function(){
              test.string(null);
            })
            .exception(function(){
              test.string(undefined);
            })
          ;
        })
      ;
```

<a name="asserter-string-assertions-of-string"></a>
## Assertions of string()
is(expected).

```js
var str = 'Hello world !';
      test
        .string(str)
          .is('Hello world !')
        .exception(function(){
          test.string(str).is('foo');
        })
      ;
```

isNot(expected).

```js
var str = 'Hello world !';
      test
        .string(str)
          .isNot('hello world !')
        .exception(function(){
          test.string(str).isNot('Hello world !');
        })
      ;
```

isIdenticalTo(expected).

```js
var str = 'Hello world !';
      test
        .string(str)
          .isIdenticalTo('Hello world !')
        .exception(function(){
          test.string(str).isIdenticalTo('Hello World !');
        })
      ;
```

isNotIdenticalTo(expected).

```js
var str = 'Hello world !';
      test
        .string(str)
          .isNotIdenticalTo('hello world !')
        .exception(function(){
          test.string(str).isNotIdenticalTo('Hello world !');
        })
      ;
```

isEqualTo(expected).

```js
var str = 'Hello world !';
      test
        .string(str)
          .isEqualTo('Hello world !')
        .exception(function(){
          test.string(str).isEqualTo('Hello World !');
        })
      ;
```

isNotEqualTo(expected).

```js
var str = 'Hello world !';
      test
        .string(str)
          .isNotEqualTo('hello world !')
        .exception(function(){
          test.string(str).isNotEqualTo('Hello world !');
        })
      ;
```

match(expected).

```js
// Assert a string value with a expected string
      test.string('Hello').match('Hello');
      // Assert a string value with a RegExp
      test.string('Hello world !').match(/world/i);
      // Assert a string with a function
      test.string('hello').match(function(it){
        return it === 'hello';
      });
      test.exception(function(){
        test.string('hello').match(/foo/);
      });
```

notMatch(expected).

```js
test
        .string('foobar')
          .notMatch('some value')
          .notMatch(/[foo]+bazzz$/)
        .string('foo')
          .notMatch(function(it){
            return it === 'bar';
          })
        .exception(function(){
          test.string('Hello Nico!').notMatch(/nico/i);
        })
      ;
```

isValid(expected).

```js
// Assert a string value with a expected string
      test.string('Hello').isValid('Hello');
      // Assert a string value with a RegExp
      test.string('Hello world !').isValid(/world/i);
      // Assert a string with a function
      test.string('hello').isValid(function(it){
        return it === 'hello';
      });
      test.exception(function(){
        test.string('hello').isValid(/foo/);
      });
```

isNotValid(expected).

```js
test
        .string('foobar')
          .isNotValid('some value')
          .isNotValid(/[foo]+bazzz$/)
        .string('foo')
          .isNotValid(function(it){
            return it === 'bar';
          })
        .exception(function(){
          test.string('Hello Nico!').notMatch(/nico/i);
        })
      ;
```

matchEach(expected).

```js
var str = 'Hello Nico!';
      test
        .string(str)
          .matchEach([/hello/i, 'Nico', function(it){
            return it === 'Hello Nico!';
          }])
        .case('Test failure', function(){
          test
            .exception(function(){
              test.string(str).matchEach([/hello/i, 'nico', function(it){
                return it === 'Hello Nico!';
              }]);
            })
            .exception(function(){
              test.string(str).matchEach([/hello/i, 'Nico', function(it){
                return it === 'Hello nico!';
              }]);
            })
          ;
        })
      ;
```

notMatchEach(expected).

```js
var str = 'Hello Nico!';
      test
        .string(str)
          .notMatchEach([/foo/i, 'bad word', function(it){
            return it === 'Bye';
          }])
        .case('Test failure', function(){
          test
            .exception(function(){
              test.string(str).notMatchEach([/hello/, 'Nico', function(it){
                return it === 'Hello !';
              }]);
            })
            .exception(function(){
              test.string(str).notMatchEach([/hello/, 'nico', function(it){
                return it === 'Hello Nico!';
              }]);
            })
          ;
        })
      ;
```

isEmpty().

```js
test
  .string('')
    .isEmpty()
  .exception(function(){
    test.string(str).isEmpty();
  })
;
```

isNotEmpty().

```js
test
  .string('a')
    .isNotEmpty()
  .exception(function(){
    test.string('').isNotEmpty();
  })
;
```

hasLength(expected).

```js
test
  .string('Hello Nico')
    .hasLength(10)
  .exception(function(){
    test.string('abc').hasLength(4);
  })
;
```

hasNotLength(expected).

```js
test
  .string('Hello Nico')
    .hasNotLength(11)
  .exception(function(){
    test.string('abc').hasNotLength(3);
  })
;
```

hasValue(expected).

```js
test
  .string('Hello, Nico!')
    .hasValue('Nico')
  .exception(function(){
    test.string('Hello').hasValue('hello');
  })
;
```

notHasValue(expected).

```js
test
  .string('Hello, Nico!')
    .notHasValue('Bye')
  .exception(function(){
    test.string('Hello').notHasValue('Hello');
  })
;
```

hasValues(expected).

```js
test
  .string('Hello Nico!')
    .hasValues(['Hello', 'Nico'])
  .exception(function(){
    test.string('Hello Nico!').hasValues(['Hi', 'Nico']);
  })
;
```

notHasValues(expected).

```js
test
  .string('Sarah Connor ?')
    .notHasValues(['next', 'door'])
  .exception(function(){
    test.string('Hello Nico!').notHasValues(['Hi', 'Nico']);
  })
;
```

contains(expected [, ...]).

```js
test
  .string('hello boy')
    .contains('boy')
  .string('KISS principle : Keep it Simple, Stupid')
    .contains('Simple', 'principle', ':')
  .exception(function(){
    test.string('Hello').contains('hello');
  })
;
```

notContains(expected [, ...]).

```js
test
  .string('hello boy')
    .notContains('bye')
  .exception(function(){
    test.string('Hello').notContains('Hello');
  })
;
```

startsWith(str).

```js
test
  .string('foobar')
    .startsWith('foo')
  .exception(function(){
    test.string('Hello the world').startsWith('world');
  })
;
```

notStartsWith(str).

```js
test
  .string('foobar')
    .notStartsWith('bar')
  .exception(function(){
    test.string('Hello the world').notStartsWith('Hello');
  })
;
```

endsWith(str).

```js
test
  .string('foobar')
    .endsWith('bar')
  .exception(function(){
    test.string('Hello the world').endsWith('Hello');
  })
;
```

notEndsWith(str).

```js
test
  .string('foobar')
    .notEndsWith('foo')
  .exception(function(){
    test.string('Hello the world').notEndsWith('world');
  })
;
```

<a name="asserter-undefined"></a>
# Asserter undefined()
<a name="asserter-undefined-undefined-behavior"></a>
## undefined() behavior
Does not contains assertions from the assertions containers.

```js
var assertions = rawAssertions.call(this, undefined);
      for(var method in assertions){
        // if is the native (inherited) method Object.hasOwnProperty
        if(method == 'hasOwnProperty'
          && test.undefined(undefined)[method] === Object.hasOwnProperty)
          continue;
        test.value(test.undefined(undefined)[method]).isUndefined();
      }
      // ensures the test method
      test.exception(function(){
        test.value(test.undefined(undefined)['hasOwnProperty']).isUndefined();
      });
```

Assert that the tested value is `undefined`.

```js
test
        .undefined(undefined)
        .undefined()
        .case('Test failure', function(){
          test
            .exception(function(){
              test.undefined(0);
            })
            .exception(function(){
              test.undefined(1);
            })
            .exception(function(){
              test.undefined('undefined');
            })
            .exception(function(){
              test.undefined(null);
            })
            .exception(function(){
              test.undefined(false);
            })
            .exception(function(){
              test.undefined(true);
            })
            .exception(function(){
              test.undefined('');
            })
            .exception(function(){
              test.undefined([]);
            })
            .exception(function(){
              test.undefined({});
            })
            .exception(function(){
              test.undefined(function(){});
            })
          ;
        })
      ;
```

<a name="asserter-value"></a>
# Asserter value()
<a name="asserter-value-value-behavior"></a>
## value() behavior
value() has all assertions.

```js
var assertions = rawAssertions.call(this, undefined);
      for(var method in assertions){
        test.value(test.value(undefined)[method]).exists();
      }
      // ensures the test method
      test.exception(function(){
        test.value(test.value(undefined)['foobar_not_exist']).exists();
      });
```

<a name="asserter-value-assertions-of-value"></a>
## Assertions of value()
is(expected).

```js
test
  .value({fluent: 'is awesome', deep: [0, 1]})
    .is({fluent: 'is awesome', deep: [0, 1]})
  .exception(function(){
    test.value({fluent: 'is awesome', deep: [0, 1]})
      .is({fluent: 'is awesome', deep: [0, 2]});
  })
;
```

isNot(expected).

```js
test
  .value({fluent: 'is awesome', deep: [0, 1]})
    .isNot({fluent: 'is awesome', deep: [0, '1']})
  .exception(function(){
    test.value({fluent: 'is awesome', deep: [0, 1]})
      .isNot({fluent: 'is awesome', deep: [0, 1]});
  })
;
```

isIdenticalTo(expected).

```js
test
  .value(1)
    .isIdenticalTo(1)
  .exception(function(){
    test.value(1).isIdenticalTo(2);
  })
;
```

isNotIdenticalTo(expected).

```js
test
  .value('1')
    .isNotIdenticalTo(1)
  .exception(function(){
    test.value(1).isNotIdenticalTo(1);
  })
;
```

isEqualTo(expected).

```js
test
  .value('1')
    .isEqualTo(1)
  .exception(function(){
    test.value(1).isEqualTo(1.1);
  })
;
```

isNotEqualTo(expected).

```js
test
  .value('foobar')
    .isNotEqualTo([])
  .exception(function(){
    test.value('foobar').isNotEqualTo('foobar');
  })
;
```

match(expected).

```js
test
  .value('foobar')
    .match(/[fo]+bar$/)
  .value(['a', 'b', 'c'])
    .match(/[a-z]/)
  .value(10)
    .match(10)
  .exception(function(){
    test.value('foobar').match('whoops');
  })
;
```

notMatch(expected).

```js
test
  .value('foobar')
    .notMatch(/[foo]+bazzz$/)
  .value(['a', 'b', 'c'])
    .notMatch(/[d-z]/)
  .value(10)
    .notMatch(8)
  .exception(function(){
    test.value('foobar').notMatch('foobar');
  })
;
```

matchEach(expected).

```js
test
  .value([10, 11, 12])
    .matchEach(function(it) {
      return it >= 10;
    })
  .exception(function(){
    // error if one or several does not match
    test.value([10, 11, 12]).matchEach(function(it) {
      return it >= 11;
    });
  })
;
```

notMatchEach(expected).

```js
test
  .value([10, 11, 12])
    .notMatchEach(function(it) {
      return it >= 13;
    })
  .exception(function(){
    // error all match
    test.value([10, 11, 12]).notMatchEach(function(it) {
      return it >= 11;
    });
  })
;
```

isValid(expected).

```js
test
  .value(42)
    .isValid(function(actual) {
      return actual === 42;
    })
  .exception(function(){
    test.value(42).isValid(function(actual) {
      return actual === 'expected value';
    });
  })
;
```

isNotValid(expected).

```js
test
  .value(42)
    .isNotValid(function(actual) {
      return actual === 44;
    })
  .exception(function(){
    test.value(42).isNotValid(function(actual) {
      return actual === 42;
    });
  })
;
```

isType(expected).

```js
test
  .value('foobar').isType('string')
  .value(0).isType('number')
  .value(1).isType('number')
  .value(1.2).isType('number')
  .value('1').isType('string')
  .value({}).isType('object')
  .value(undefined).isType('undefined')
  .value(null).isType('object')
  .value(true).isType('boolean')
  .value(false).isType('boolean')
  .exception(function(){
    test.value('1').isType('number');
  })
;
```

isNotType(expected).

```js
test
  .value({}).isNotType('string')
  .value('0').isNotType('number')
  .value('1').isNotType('number')
  .value('1.2').isNotType('number')
  .value(1).isNotType('string')
  .value(null).isNotType('undefined')
  .value(undefined).isNotType('object')
  .value('true').isNotType('boolean')
  .value('false').isNotType('boolean')
  .exception(function(){
    test.value('1').isNotType('string');
  })
;
```

isObject().

```js
test
  .value({})
    .isObject()
  .exception(function(){
    test.value(function(){}).isObject();
  })
;
```

isArray().

```js
test
  .value([])
    .isArray()
  .exception(function(){
    test.value({}).isArray();
  })
;
```

isFunction().

```js
test
  .value(function(){})
    .isFunction()
  .exception(function(){
    test.value(new Date()).isFunction();
  })
;
```

isString().

```js
test
  .value('foobar')
    .isString()
  .exception(function(){
    test.value(10.2).isString();
  })
;
```

isNumber().

```js
test
  .value(42)
    .isNumber()
  .exception(function(){
    test.value(true).isNumber();
  })
;
```

isBool() - alias of isBoolean().

```js
test
  .value(false)
    .isBool()
  .exception(function(){
    test.value(null).isBool();
  })
;
```

isBoolean().

```js
test
  .value(true)
    .isBoolean()
  .exception(function(){
    test.value(undefined).isBoolean();
  })
;
```

isNull().

```js
test
  .value(null)
    .isNull()
  .exception(function(){
    test.value(false).isNull();
  })
  .exception(function(){
    test.value(undefined).isNull();
  })
  .exception(function(){
    test.value(0).isNull();
  })
;
```

isUndefined().

```js
test
  .value(undefined)
    .isUndefined()
  .exception(function(){
    test.value(null).isUndefined();
  })
;
```

isRegExp().

```js
test
  .value(/[0-9]+/)
    .isRegExp()
  .exception(function(){
    test.value('/[0-9]+/').isRegExp();
  })
;
```

isNotRegExp().

```js
test
  .value(new Date())
    .isNotRegExp()
  .exception(function(){
    test.value(/foo/).isNotRegExp();
  })
;
```

isDate().

```js
test
  .value(new Date())
    .isDate()
  .exception(function(){
    test.value({month:5, year:2012, day:12}).isDate();
  })
;
```

isNotDate().

```js
test
  .value(/[0-9]+/)
    .isNotDate()
  .exception(function(){
    test.value(new Date()).isNotDate();
  })
;
```

isArguments().

```js
var fn = function(){
        test.value(arguments).isArguments();
      };
      fn(1, 2, 3);
      test.exception(function(){
        test.value({0: 'a'}).isArguments();
      });
```

isNotArguments().

```js
var fn = function(){
        test
          .value([1, 2, 3])
            .isNotArguments()
          .value({0:1, 1:2, 2:3})
            .isNotArguments()
          .exception(function(){
            test.value(arguments).isNotArguments();
          })
        ;
      };
      fn(1, 2, 3);
```

isTrue().

```js
test
  .value(true)
    .isTrue()
  .exception(function(){
    test.value(1).isTrue();
  })
;
```

isNotTrue().

```js
test
        .value(false)
          .isNotTrue()
        .value(1)
          .isNotTrue()
        .value('1')
          .isNotTrue()
        .value('true')
          .isNotTrue()
        .exception(function(){
          test.value(true).isNotTrue();
        })
      ;
```

isTruthy().

```js
test
  .value(true)
    .isTruthy()
  .value(1)
    .isTruthy()
  .value('ok')
    .isTruthy()
  .exception(function(){
    test.value('').isTruthy();
  })
;
```

isNotTruthy().

```js
test
  .value(0)
    .isNotTruthy()
  .exception(function(){
    test.value('1').isNotTruthy();
  })
;
```

isFalse().

```js
test
  .value(false)
    .isFalse()
  .exception(function(){
    test.value(null).isFalse();
  })
;
```

isNotFalse().

```js
test
        .value(true)
          .isNotFalse()
        .value(0)
          .isNotFalse()
        .value('0')
          .isNotFalse()
        .value('false')
          .isNotFalse()
        .value(null)
          .isNotFalse()
        .value(undefined)
          .isNotFalse()
        .exception(function(){
          test.value(false).isNotFalse();
        })
      ;
```

isFalsy().

```js
test
  .value(false)
    .isFalsy()
  .value(0)
    .isFalsy()
  .value('')
    .isFalsy()
  .value(null)
    .isFalsy()
  .value(undefined)
    .isFalsy()
  .exception(function(){
    test.value(1).isFalsy();
  })
;
```

isNotFalsy().

```js
test
  .value(1)
    .isNotFalsy()
  .exception(function(){
    test.value(undefined).isNotFalsy();
  })
;
```

isEmpty().

```js
test
  .value('')
    .isEmpty()
  .value([])
    .isEmpty()
  .value({})
    .isEmpty()
  .exception(function(){
    test.value(1).isEmpty();
  })
;
```

isNotEmpty().

```js
test
  .value('a')
    .isNotEmpty()
  .exception(function(){
    test.value('').isNotEmpty();
  })
  .exception(function(){
    test.value({}).isNotEmpty();
  })
;
```

isNaN().

```js
test
  .value(NaN)
    .isNaN()
  .value(new Number(NaN))
    .isNaN()
  .value(0/0)
    .isNaN()
  .value(parseInt('a', 10))
    .isNaN()
  .exception(function(){
    test.value(1).isNaN();
  })
  .exception(function(){
    test.value(new Number(1)).isNaN();
  })
  .exception(function(){
    test.value(undefined).isNaN();
  })
  .exception(function(){
    test.value(null).isNaN();
  })
  .exception(function(){
    test.value(false).isNaN();
  })
  .exception(function(){
    test.value(true).isNaN();
  })
  .exception(function(){
    test.value(0).isNaN();
  })
  .exception(function(){
    test.value({}).isNaN();
  })
  .exception(function(){
    test.value([]).isNaN();
  })
;
```

isNotNaN().

```js
test
  .value(1)
    .isNotNaN()
  .value(new Number(1))
    .isNotNaN()
  .value(undefined)
    .isNotNaN()
  .value(null)
    .isNotNaN()
  .value(false)
    .isNotNaN()
  .value(true)
    .isNotNaN()
  .value(0)
    .isNotNaN()
  .value({})
    .isNotNaN()
  .value([])
    .isNotNaN()
  .exception(function(){
    test.value(NaN).isNotNaN();
  })
  .exception(function(){
    test.value(new Number(NaN)).isNotNaN();
  })
  .exception(function(){
    test.value(0/0).isNotNaN();
  })
  .exception(function(){
    test.value(parseInt('a', 10)).isNotNaN();
  })
;
```

exists().

```js
test
  .value('foobar')
    .exists()
  .exception(function(){
    test.value(null).exists();
  })
;
```

isError() - alias of throws(Error).

```js
var trigger = function(){
        throw new Error('Whoops!');
      };
      test
        .value(trigger)
          .isError()
        .case('Test failure', function(){
          test
            .value(function(){
              test.value(function(){
                throw {name: 'error', message: 'Whoops'};
              })
              .isError();
            })
            .throws()
            .value(function(){
              test.value(function(){
                throw Error; // <= not instanciated
              })
              .isError();
            })
            .throws()
          ;
        })
      ;
```

throws([constructor], [expected]).

```js
var indicator, trigger = function(){
        throw new Error("I'm a ninja !");
      };
      test
        .value(trigger)
          .throws()
          .throws("I'm a ninja !")
          .throws(/ninja/)
          .throws(Error)
          .throws(Error, "I'm a ninja !")
          .throws(Error, /ninja/)
          .throws(function(err) {
            if ((err instanceof Error) && /ninja/.test(err) ) {
              indicator = true; // just for test and example
              return true;
            }
          })
        // 'then' does nothing, it's just to make the test more expressive
        .then()
          .bool(indicator)
            .isTrue()
        .exception(function(){
          test.value(function(){
            return true;
          })
          .throws();
        })
      ;
```

hasLength(expected).

```js
test
  .value([1, 2])
    .hasLength(2)
  .value('Hello Nico')
    .hasLength(10)
  .exception(function(){
    test.value('Hello Nico').hasLength(2);
  })
;
```

hasNotLength(expected).

```js
test
  .value([1, 2])
    .hasNotLength(1)
  .value('Hello Nico')
    .hasNotLength(11)
  .exception(function(){
    test.value('Hello Nico').hasNotLength(10);
  })
;
```

isBetween(begin, end).

```js
test
  .value(2)
    .isBetween(2, 4)
  .value(3)
    .isBetween(2, 4)
  .value(4)
    .isBetween(2, 4)
  .exception(function(){
    test.value(2).isBetween(4, 2);
  })
;
```

isNotBetween(begin, end).

```js
test
  .value(1)
    .isNotBetween(2, 4)
  .value(5)
    .isNotBetween(2, 4)
  .exception(function(){
    test.value(2).isNotBetween(2, 2);
  })
;
```

isBefore(expected).

```js
test
  .value(new Date(2010, 5, 20))
    .isBefore(new Date(2012, 2, 28))
  .exception(function(){
    test.value(new Date(2012, 2, 28))
      .isBefore(new Date(1982, 2, 17));
  })
;
```

isAfter(expected).

```js
test
  .value(new Date(2012, 2, 28))
    .isAfter(new Date(2010, 5, 20))
  .exception(function(){
    test.value(new Date(2012, 2, 28))
      .isAfter(new Date(2014, 2, 28));
  })
;
```

isLessThan(expected).

```js
test
  .value(1)
    .isLessThan(2)
  .exception(function(){
    test.value(1).isLessThan(0.98);
  })
;
```

isGreaterThan(expected).

```js
test
  .value(2)
    .isGreaterThan(1)
  .exception(function(){
    test.value(1).isGreaterThan(1.00000001);
  })
;
```

isApprox(num, delta).

```js
test
  .value(99.98)
    .isApprox(100, 0.1)
  .exception(function(){
    test.value(99.98)
      .isApprox(100, 0.01);
  })
;
```

isInfinite().

```js
test
  .value(1/0)
    .isInfinite()
  .exception(function(){
    test.value(1.333333333333333333333)
      .isInfinite();
  })
;
```

isNotInfinite().

```js
test
  .value(1.3333333333333333333333333)
    .isNotInfinite()
  .exception(function(){
    test.value(1/0)
      .isNotInfinite();
  })
;
```

isEnumerable(property).

```js
test
  .value({prop: 'foobar'})
    .isEnumerable('prop')
  .exception(function(){
    test.value(function() {})
      .isEnumerable('call');
  })
;
```

isNotEnumerable(property).

```js
test
  .value(function() {})
    .isNotEnumerable('call')
  .value(Object.create({}, {prop: {enumerable: 0}}))
    .isNotEnumerable('prop')
  .exception(function(){
    test.value({prop: 'foobar'})
      .isNotEnumerable('prop');
  })
;
```

isFrozen().

```js
test
  .value(Object.freeze({}))
    .isFrozen()
  .exception(function(){
    test.value({})
      .isFrozen();
  })
;
```

isNotFrozen().

```js
test
  .value({})
    .isNotFrozen()
  .exception(function(){
    test.value(Object.freeze({}))
      .isNotFrozen();
  })
;
```

isInstanceOf(expected).

```js
test
  .value(new Date())
    .isInstanceOf(Date)
  .exception(function(){
    test.value(new Date())
      .isInstanceOf(Array);
  })
;
```

isNotInstanceOf(expected).

```js
test
  .value(new Date())
    .isNotInstanceOf(RegExp)
  .exception(function(){
    test.value(new Date())
      .isNotInstanceOf(Object);
  })
;
```

hasProperty(property [, value]).

```js
test
  .value({foo: 'bar'})
    .hasProperty('foo')
  .value({foo: 'bar'})
    .hasProperty('foo', 'bar')
  .exception(function(){
    test.value({})
      .hasProperty('foo');
  })
;
```

hasNotProperty(property [, value]).

```js
test
  .value({foo: 'bar'})
    .hasNotProperty('bar')
  .value({foo: 'bar'})
    .hasNotProperty('foo', 'baz')
  .exception(function(){
    test.value({foo: 'bar'})
      .hasNotProperty('foo');
  })
;
```

hasOwnProperty(property [, value]).

```js
test
  .value({foo: 'bar'})
    .hasOwnProperty('foo')
  .value({foo: 'bar'})
    .hasOwnProperty('foo', 'bar')
  .exception(function(){
    test.value(new RegExp('foo'))
      .hasOwnProperty('constructor');
  })
;
```

hasNotOwnProperty(property [, value]).

```js
test
  .value({foo: 'bar'})
    .hasNotOwnProperty('bar')
  .value({foo: 'bar'})
    .hasNotOwnProperty('foo', 'baz')
  .exception(function(){
    test.value({foo: 'bar'})
      .hasNotOwnProperty('foo');
  })
;
```

hasProperties(properties).

```js
var obj = {foo: 'bar', bar: 'huhu', other: 'vroom'};
      test
        .value(obj)
          .hasProperties(['other', 'bar', 'foo'])
        .exception(function(){
          test.value(obj)
            .hasProperties(['other', 'bar']);
        })
      ;
```

hasNotProperties(properties).

```js
var obj = {foo: 'bar', bar: 'huhu', other: 'vroom'};
      test
        .value(obj)
          .hasNotProperties(['other', 'foo'])
        .exception(function(){
          test.value(obj)
            .hasNotProperties(['other', 'bar', 'foo']);
        })
      ;
```

hasOwnProperties(properties).

```js
var obj = {foo: 'bar', bar: 'huhu', other: 'vroom'};
      test
        .value(obj)
          .hasOwnProperties(['other', 'bar', 'foo'])
        .exception(function(){
          test.value(obj)
            .hasOwnProperties(['other', 'bar']);
        })
      ;
```

hasKey(key [, value]).

```js
test
  .value({foo: 'bar'})
    .hasKey('foo')
  .value({foo: 'bar'})
    .hasKey('foo', 'bar')
  .exception(function(){
    test.value({})
      .hasKey('foo');
  })
;
```

notHasKey(key [, value]).

```js
test
  .value({foo: 'bar'})
    .notHasKey('bar')
  .value({foo: 'bar'})
    .notHasKey('foo', 'baz')
  .exception(function(){
    test.value({foo: 'bar'})
      .notHasKey('foo');
  })
;
```

hasKeys(keys).

```js
var obj = {foo: 'bar', bar: 'huhu', other: 'vroom'};
      test
        .value(obj)
          .hasKeys(['other', 'bar', 'foo'])
        .exception(function(){
          test.value(obj)
            .hasKeys(['other', 'bar']);
        })
      ;
```

notHasKeys(keys).

```js
var obj = {foo: 'bar', bar: 'huhu', other: 'vroom'};
      test
        .value(obj)
          .notHasKeys(['other', 'foo'])
        .exception(function(){
          test.value(obj)
            .notHasKeys(['other', 'bar', 'foo']);
        })
      ;
```

hasValue(expected).

```js
test
  .value('Hello, Nico!')
    .hasValue('Nico')
  .value([1, 42, 3])
    .hasValue(42)
  .value({life: 42, love: 69})
    .hasValue(42)
  .exception(function(){
    test.value({life: 42, love: 69})
      .hasValue(6);
  })
;
```

notHasValue(expected).

```js
test
  .value('Hello, Nico!')
    .notHasValue('Bye')
  .value([1, 42, 3])
    .notHasValue(4)
  .value({life: 42, love: 69})
    .notHasValue(4)
  .exception(function(){
    test.value({life: 42, love: 69})
      .notHasValue(69);
  })
;
```

hasValues(expected).

```js
test
  .value([1, 42, 3])
    .hasValues([42, 3])
  .exception(function(){
    test.value([1, 42, 3])
      .hasValues([42, 3.01]);
  })
;
```

notHasValues(expected).

```js
test
  .value([1, 42, 3])
    .notHasValues([4, 2])
  .exception(function(){
    test.value([1, 42, 3])
      .notHasValues([1, 42, 3]);
  })
;
```

contains(expected [, ...]).

```js
test
  .value('hello boy')
    .contains('boy')
  .value('KISS principle : Keep it Simple, Stupid')
    .contains('Simple', 'principle', ':')
  .value([1,2,3])
    .contains([3])
  .value([1,2,3])
    .contains([1, 3])
  .value([1,2,3])
    .contains([3, 1, 2])
  .value({ a: { b: 10 }, b: { c: 10, d: 11, a: { b: 10, c: 11} }})
    .contains({ a: { b: 10 }, b: { c: 10, a: { c: 11 }}})
  .value([1, 2, 3, { a: { b: { d: 12 }}}])
    .contains([2], [1, 2], [{ a: { b: {d: 12}}}])
  .value([[1],[2],[3]])
    .contains([[3]])
  .value([[1],[2],[3, 4]])
    .contains([[3]])
  .value([{a: 'a'}, {b: 'b', c: 'c'}])
    .contains([{a: 'a'}], [{b: 'b'}])
  .exception(function(){
    test.value([{a: 'a'}, {b: 'b', c: 'c'}])
    .contains([{a: 'a'}], [{b: 'c'}]);
  })
;
```

notContains(expected [, ...]).

```js
test
  .value('hello boy')
    .notContains('bye')
  .value([[1],[2],[3, 4]])
    .notContains([[0]])
  .value([{a: 'a'}, {b: 'b', c: 'c'}])
    .notContains([{a: 'b'}], [{c: 'b'}])
  .exception(function(){
    test.value([{a: 'a'}, {b: 'b', c: 'c'}])
      .notContains([{a: 'a'}, {c: 'c'}]);
  })
;
```

isReverseOf(expected).

```js
test
        .value([1, 2, 3])
          .isReverseOf([3, 2, 1])
        .exception(function() {
          test.value([1, 2, 3])
            .isReverseOf([1, 2, 3]);
        })
        .exception(function() {
          test.value([1, 2, 3])
            .isReverseOf([3, 2, 2, 1]);
        })
      ;
```

isNotReverseOf(expected).

```js
test
        .value([1, 2, 2, 3])
          .isNotReverseOf([3, 2, 1])
        .exception(function() {
          test.value([3, 2, 1])
            .isNotReverseOf([3, 2, 1]);
        })
      ;
```

startsWith(str).

```js
test
  .value('foobar')
    .startsWith('foo')
  .exception(function(){
    test.value('Hello the world').startsWith('world');
  })
;
```

notStartsWith(str).

```js
test
  .value('foobar')
    .notStartsWith('bar')
  .exception(function(){
    test.value('Hello the world').notStartsWith('Hello');
  })
;
```

endsWith(str).

```js
test
  .value('foobar')
    .endsWith('bar')
  .exception(function(){
    test.value('Hello the world').endsWith('Hello');
  })
;
```

notEndsWith(str).

```js
test
  .value('foobar')
    .notEndsWith('foo')
  .exception(function(){
    test.value('Hello the world').notEndsWith('world');
  })
;
```

hasHttpStatus(code).

```js
var req = {
        headers: {
          'content-type': 'application/json',
        },
        statusCode: 200
      };
      test
        .value(req)
          .hasHttpStatus(200)
        .value(req)
          .hasProperty('statusCode', 200)
        .exception(function(){
          test.value(req).hasHttpStatus(500);
        })
      ;
```

notHasHttpStatus(code).

```js
var req = {
        headers: {
          'content-type': 'application/json',
        },
        'statusCode': 200
      };
      test
        .value(req)
          .notHasHttpStatus(404)
        .value(req)
          .hasNotProperty('statusCode', 404)
        .exception(function(){
          test.value(req).notHasHttpStatus(200);
        })
      ;
```

hasHeader(field [, value]).

```js
var req = {
        headers: {
          'content-type': 'application/json'
        }
      };
      test
        .exception(function(){
          test.value(req).hasHeader('charset');
        })
        .exception(function(){
          test.value(req).hasHeader('content-type', 'text/html');
        })
        .value(req)
          .hasHeader('content-type')
          .hasHeader('content-type', 'application/json')
      ;
      // or
      test
        .object(req.headers)
          .hasProperty('content-type')
        .string(req.headers['content-type'])
          .startsWith('application/json')
      ;
      // or
      test
        .string(req.headers['content-type'])
          .isNotEmpty()
        .string(req.headers['content-type'])
          .startsWith('application/json')
      ;
```

notHasHeader(field [, value]).

```js
var req = {
        headers: {
          'content-type': 'application/json'
        }
      };
      test
        .exception(function(){
          test.value(req).notHasHeader('content-type');
        })
        .exception(function(){
          test.value(req).notHasHeader('content-type', 'application/json');
        })
        .value(req)
          .notHasHeader('charset')
          .notHasHeader('content-type', 'text/html')
        // other test cases
        .value({})
          .notHasHeader('charset')
          .notHasHeader('content-type', 'text/html')
        .value({headers: {}})
          .notHasHeader('charset')
          .notHasHeader('content-type', 'text/html')
      ;
```

hasHeaderJson().

```js
var req = {
  headers: {
    'content-type': 'application/json'
  }
};
test.value(req).hasHeaderJson();
// or
test
  .string(req.headers['content-type'])
    .startsWith('application/json')
;
test
  .then(req.headers['content-type'] = 'application/json; charset=utf-8')
  .value(req).hasHeaderJson()
;
// or
test
  .string(req.headers['content-type'])
    .startsWith('application/json')
;
test
  .then(req.headers['content-type'] = 'text/html')
  .error(function(){
    test.value(req).hasHeaderJson();
  })
;
```

notHasHeaderJson().

```js
var req = {
        headers: {
          'content-type': 'text/html'
        }
      };
      test
        .value(req)
          .notHasHeaderJson()
        .value({})
          .notHasHeaderJson()
      ;
      // or
      test
        .value(req)
          .notHasHeaderJson()
        .string(req.headers['content-type'])
          .notStartsWith('application/json')
        .value({})
          .notHasHeaderJson()
        .then(req.headers['content-type'] = 'application/json')
        .error(function(){
          test.value(req).notHasHeaderJson();
        })
      ;
```

hasHeaderHtml().

```js
var req = {
        headers: {
          'content-type': 'text/html'
        }
      };
      test.value(req).hasHeaderHtml();
      // or
      test.string(req.headers['content-type']).startsWith('text/html');
      test
        .then(req.headers['content-type'] = 'text/html; charset=utf-8')
        .value(req).hasHeaderHtml()
      ;
      // or
      test.string(req.headers['content-type']).startsWith('text/html');
      test
        .then(req.headers['content-type'] = 'application/json')
        .error(function(){
          test.value(req).hasHeaderHtml();
        })
      ;
```

notHasHeaderHtml().

```js
var req = {
        headers: {
          'content-type': 'application/json'
        }
      };
      test
        .value(req)
          .notHasHeaderHtml()
      // or
        .string(req.headers['content-type'])
          .notStartsWith('text/html')
        .then(req.headers['content-type'] = 'text/html')
        .error(function(){
          test.value(req).notHasHeaderHtml();
        })
      ;
```

<a name="unitjs-provides-several-api-unified-and-several-assertion-styles"></a>
# Unit.js provides several API unified and several assertion styles
Unit.js style.

```js
// test 'string' type
    test.string('foobar')
      // then that actual value '==' expected value
      .isEqualTo('foobar')
      // then that actual value '===' expected value
      .isIdenticalTo('foobar')
    ;
```

Assert library of Node.js.

```js
// test 'string' type
    test.assert(typeof 'foobar' == 'string');
    // then that actual value '==' expected value
    test.assert.equal('foobar', 'foobar');
    // then that actual value '===' expected value
    test.assert.strictEqual('foobar', 'foobar');
    // this shortcut works also like this
    var assert = test.assert;
    // test 'string' type
    assert(typeof 'foobar' == 'string');
    // then that actual value '==' expected value
    assert.equal('foobar', 'foobar');
    // then that actual value '===' expected value
    assert.strictEqual('foobar', 'foobar');
```

Should.js library.

```js
// test 'string' type
    test.should('foobar').be.type('string');
    // then that actual value '==' expected value
    test.should('foobar' == 'foobar').be.ok;
    // then that actual value '===' expected value
    test.should('foobar').be.equal('foobar');
    // Should.js library (alternative style)
    var should = test.should;
    // test 'string' type
    ('foobar').should.be.type('string');
    // then that actual value '==' expected value
    ('foobar' == 'foobar').should.be.ok;
    // then that actual value '===' expected value
    ('foobar').should.be.equal('foobar');
    // this shortcut works also like this
    // test 'string' type
    should('foobar').be.type('string');
    // then that actual value '==' expected value
    should('foobar' == 'foobar').be.ok;
    // then that actual value '===' expected value
    should('foobar').be.equal('foobar');
```

Must.js library.

```js
// test 'string' type
    test.must('foobar').be.a.string();
    // then that actual value '==' expected value
    test.must('foobar' == 'foobar').be.true();
    // then that actual value '===' expected value
    test.must('foobar').be.equal('foobar');
    // Must.js library (alternative style)
    var must = test.must;
    // test 'string' type
    ('foobar').must.be.a.string();
    // then that actual value '==' expected value
    ('foobar' == 'foobar').must.be.true();
    // then that actual value '===' expected value
    ('foobar').must.be.equal('foobar');
    // this shortcut works also like this
    // test 'string' type
    must('foobar').be.a.string();
    // then that actual value '==' expected value
    must('foobar' == 'foobar').be.true();
    // then that actual value '===' expected value
    must('foobar').be.equal('foobar');
```

Unit.js is expressive and fluent.

```js
var
      obj = {
        fluent: 'is awesome'
      },
      copy_obj = obj,
      num,
      arr;
    test
      .object(obj)
        // ===
        // passes because is the same object
        .isIdenticalTo(copy_obj)
        // !==
        // passes because is not the same object
        .isNotIdenticalTo({
          fluent: 'is awesome'
        })
        // !=
        // passes because is not the same object
        .isNotEqualTo({
          fluent: 'is awesome'
        })
        // ==
        // passes because 'is' check that the objects are the same structure
        .is({
          fluent: 'is awesome'
        })
        .hasProperty('fluent', 'is awesome')
      .given(num = 10)
      .number(num)
        .isBetween(5, 15)
      .then()
        .error(function() {
          test.number(num / 1).isInfinite();
        })
      .if(num = num * 0.4)
      .number(num)
        // 10 * 0.4 = 4, ok 4 is approximately 5 (with a delta of 1.5)
        .isApprox(5, 1.5)
      .when(function() {
        arr = [];
        arr.push(num);
        arr.push('fluent');
      })
      .array(arr)
        .hasLength(2)
        .hasValue('fluent')
      .if(arr.push('test flow'))
      .and(arr.push('is expressive'))
      .array(arr)
        .hasLength(4)
        // provides expected via an array
        .hasValues(['fluent', 'test flow', 'is expressive'])
    ;
```

<a name="control-flow"></a>
# Control flow
<a name="control-flow-performs-the-tests-without-entangled-with-the-flow-of-other-series-of-tests"></a>
## Performs the tests without entangled with the flow of other series of tests
Series launched with chaining.

```js
test
  .exception(function() {
    test
      .string('serie1-1')
        .isEqualTo('serie1-1')
      .string('serie1-2')
        .isEqualTo('serie1-2')
      .string('serie1-3')
        .isEqualTo('serie1-3')
    ;
    throw new Error('Whoops1-1 !');
  })
    .hasMessage('Whoops1-1 !')
    .hasMessage(/Whoop/)
  .string('serie2-1')
    .isEqualTo('serie2-1')
  .exception(function() {
    test
      .string('serie2-1')
        .isEqualTo('serie2-1')
      .string('serie2-2')
        .isEqualTo('serie2-2')
      .string('serie2-3')
        .isEqualTo('serie2-3')
    ;
    throw new Error('Whoops2-1 !');
  })
    .hasMessage('Whoops2-1 !')
    .hasMessage(/Whoop/)
  .string('serie3-1')
    .isEqualTo('serie3-1')
  .exception(function() {
    throw new Error('Whoops3-1 !');
  })
    .hasMessage('Whoops3-1 !')
  .string('serie4-1')
    .isEqualTo('serie4-1')
  .exception(function() {
    throw new Error('Whoops4-1 !');
  })
    .hasMessage('Whoops4-1 !')
  .string('serie5-1')
    .isEqualTo('serie5-1')
  .string('serie5-2')
    .isEqualTo('serie5-2')
  .string('serie5-3')
    .isEqualTo('serie5-3')
  .then()
    .string('serie5-4')
    .isEqualTo('serie5-4')
;
```

Series launched without chaining.

```js
test
  .exception(function() {
    test.string('serie1-1').isEqualTo('serie1-1');
    test.string('serie1-2').isEqualTo('serie1-2');
    test.string('serie1-3').isEqualTo('serie1-3');
    throw new Error('Whoops1-1 !');
  })
    .hasMessage('Whoops1-1 !')
    .hasMessage(/Whoop/)
;
test.string('serie2-1').isEqualTo('serie2-1');
test
  .exception(function() {
    test.string('serie2-1').isEqualTo('serie2-1');
    test.string('serie2-2').isEqualTo('serie2-2');
    test.string('serie2-3').isEqualTo('serie2-3');
    throw new Error('Whoops2-1 !');
  })
    .hasMessage('Whoops2-1 !')
    .hasMessage(/Whoop/);
test.string('serie3-1').isEqualTo('serie3-1');
test.exception(function() {
  throw new Error('Whoops3-1 !');
})
  .hasMessage('Whoops3-1 !');
test.string('serie4-1').isEqualTo('serie4-1');
test.exception(function() {
  throw new Error('Whoops4-1 !');
})
  .hasMessage('Whoops4-1 !');
test.string('serie5-1').isEqualTo('serie5-1');
test.string('serie5-2').isEqualTo('serie5-2');
test.string('serie5-3').isEqualTo('serie5-3');
test.string('serie5-4').isEqualTo('serie5-4');
```

Chaining with dependency injection.

```js
test
        .$provider('getStrProvider', function(self) {
          test.object(self)
            .isIdenticalTo(test.$di._container);
          // test reference
          self.fromSelf = 'ok from self';
          return function(val) {
            test.string(val);
            return val;
          };
        })
        .case('Test the chain of provider', function() {
          var fn = test.$di.get('getStrProvider');
          test
            .function(fn)
            .string(fn('ok1'))
              .isIdenticalTo('ok1')
            .string(test.$di.get('fromSelf'))
              .isIdenticalTo('ok from self')
          ;
        })
          .$invoke('getStrProvider', function(getStr) {
            test.string(getStr('ok2'))
              .isIdenticalTo('ok2');
            return test;
          })
        .case('Test the chain of a factorized function')
          .$factory('chainStrTest', 'getStrProvider', function(getStr) {
            return test
              .function(getStr)
              .string(getStr('from factory'))
            ;
          })
          .$di.get('chainStrTest')
            .isIdenticalTo('from factory')
      ;
```

Series with helpers.

```js
var str;
      var indicator;
      var log = console.log;
      console.log = test.spy();
      test
        .given(str = 'serie1-1')
        .string(str)
          .isEqualTo('serie1-1')
        .when('change "str" value', function() {
          str = 'serie1-2';
        })
        .then(function() {
          indicator = true;
          test.string(str).isEqualTo('serie1-2');
        })
        .case('Checks that "then" execute the function argument')
        .bool(indicator)
          .isTrue()
        .string('1').dump('test.dump()').is('1')
        .bool(console.log.called)
          .isTrue()
        .then(function() {
          console.log = log;
        })
        .then('Check break of flow')
        .exception(function() {
          test.string('1').case().is('1');
        })
        .exception(function() {
          test.string('1').if().is('1');
        })
        .exception(function() {
          test.string('1').and().is('1');
        })
        .exception(function() {
          test.string('1').case.is('1');
        })
        .exception(function() {
          test.string('1').if.is('1');
        })
        .exception(function() {
          test.string('1').and.is('1');
        })
        .exception(function() {
          test.string('1').given().is('1');
        })
        .exception(function() {
          test.string('1').when().is('1');
        })
        .exception(function() {
          test.string('1').then().is('1');
        })
        .then('The context of flow', function() {
          var flow = test.string('1').bool(true).isTrue();
          test
            .object(flow)
              .hasProperty('isTrue')
              .hasProperty('isFalse')
              .hasNotProperty('hasValue')
            .object(test.string('1'))
              .hasProperty('hasValue')
          ;
        })
      ;
```

<a name="passes-ioc-container"></a>
# Passes IOC container
case().

```js
test
      .case(function() {
        test.object(this)
          .isIdenticalTo(test.$di._container);
        this.callSpy();
      })
      .$di.get('assertSpy')(1)
      .then('control flow')
        .object({})
        .case(function() {
          test.object(this)
            .isIdenticalTo(test.$di._container);
          this.callSpy();
        })
        .$di.get('assertSpy')(2)
    ;
```

given().

```js
test
      .given(function() {
        test.object(this)
          .isIdenticalTo(test.$di._container);
        this.callSpy();
      })
      .$di.get('assertSpy')(1)
      .then('control flow')
        .object({})
        .given(function() {
          test.object(this)
            .isIdenticalTo(test.$di._container);
          this.callSpy();
        })
        .$di.get('assertSpy')(2)
    ;
```

when().

```js
test
      .when(function() {
        test.object(this)
          .isIdenticalTo(test.$di._container);
        this.callSpy();
      })
      .$di.get('assertSpy')(1)
      .then('control flow')
        .object({})
        .when(function() {
          test.object(this)
            .isIdenticalTo(test.$di._container);
          this.callSpy();
        })
        .$di.get('assertSpy')(2)
    ;
```

then().

```js
test
      .then(function() {
        test.object(this)
          .isIdenticalTo(test.$di._container);
        this.callSpy();
      })
      .$di.get('assertSpy')(1)
      .then('control flow')
        .object({})
        .then(function() {
          test.object(this)
            .isIdenticalTo(test.$di._container);
          this.callSpy();
        })
        .$di.get('assertSpy')(2)
    ;
```

if().

```js
test
      .if(function() {
        test.object(this)
          .isIdenticalTo(test.$di._container);
        this.callSpy();
      })
      .$di.get('assertSpy')(1)
      .then('control flow')
        .object({})
        .if(function() {
          test.object(this)
            .isIdenticalTo(test.$di._container);
          this.callSpy();
        })
        .$di.get('assertSpy')(2)
    ;
```

and().

```js
test
      .and(function() {
        test.object(this)
          .isIdenticalTo(test.$di._container);
        this.callSpy();
      })
      .$di.get('assertSpy')(1)
      .then('control flow')
        .object({})
        .and(function() {
          test.object(this)
            .isIdenticalTo(test.$di._container);
          this.callSpy();
        })
        .$di.get('assertSpy')(2)
    ;
```

wait().

```js
var spy = test.spy(function() {
      calledAt  = new Date();
      calledAt  = calledAt.getTime();
    });
    var defAt = new Date();
    var calledAt;
    test.wait(20, spy);
    setTimeout(function(){
      test.assert(spy.calledOnce);
      defAt = defAt.getTime();
      test.number(calledAt)
        .isGreaterThan(defAt + 18);
      done();
    }, 25);
```

dump().

```js
var log = console.log;
    console.log = test.spy();
    test
      .string('1')
        .dump('test.dump()')
        .is('1')
      .bool(console.log.called)
        .isTrue()
    ;
    console.log = log;
```

stats.

```js
var countAssert, countAssertOk, total;
test
  .object(test.stats)
  .object(test.stats.assertions)
  .object(test.stats.total)
  .number(test.stats.total.assertions)
    .isGreaterThan(2)
  .number(test.stats.assertions.isObject)
    .isGreaterThan(1)
  .number(test.stats.assertions.isNumber)
    .isGreaterThan(1)
  .case('assert', function() {
    total         = test.stats.total.assertions;
    countAssert   = test.stats.assertions.assert || 0;
    countAssertOk = test.stats.assertions['assert.ok'] || 0;
    test.assert(true);
    test.assert.ok(true);
    test
      .number(test.stats.total.assertions)
        .is(total + 2)
      .number(test.stats.assertions.assert)
        .is(countAssert + 1)
      .number(test.stats.assertions['assert.ok'])
        .is(countAssertOk + 1)
    ;
  })
;
```

<a name="unitjs-provides-sinonjs"></a>
# Unit.js provides sinon.js
<a name="unitjs-provides-sinonjs-spies"></a>
## Spies
calls the original function.

```js
var callback = test.spy();
var proxy = once(callback);
proxy();
test.assert(callback.called);
```

calls the original function only once.

```js
var callback = test.spy();
var proxy = once(callback);
proxy();
proxy();
test.assert(callback.calledOnce);
// ...or:
test.assert.strictEqual(callback.callCount, 1);
```

calls original function with right this and args.

```js
var callback = test.spy();
var proxy = once(callback);
var obj = {};
proxy.call(obj, 1, 2, 3);
test.assert(callback.calledOn(obj));
test.assert(callback.calledWith(1, 2, 3));
```

<a name="unitjs-provides-sinonjs-stubs"></a>
## Stubs
returns the return value from the original function.

```js
var callback = test.stub().returns(42);
var proxy = once(callback);
test.assert.strictEqual(proxy(), 42);
```

<a name="unitjs-provides-sinonjs-testing-ajax"></a>
## Testing Ajax
makes a GET request for todo items.

```js
test.stub(jQuery, 'ajax');
getTodos(42, test.spy());
test.assert(jQuery.ajax.calledWithMatch({
  url: '/todo/42/items'
}));
```

<a name="unitjs-provides-sinonjs-mocks"></a>
## Mocks
returns the return value from the original function.

```js
var myAPI = {
  method: function() {}
};
var mock = test.mock(myAPI);
mock.expects('method').once().returns(42);
var proxy = once(myAPI.method);
test.assert.equal(proxy(), 42);
mock.verify();
```

test should call a method with exceptions.

```js
var myAPI = {
  method: function() {}
};
var mock = test.mock(myAPI);
mock.expects('method').once().throws();
test.exception(function() {
  myAPI.method();
})
  .isInstanceOf(Error);
mock.verify();
```

<a name="unitjs-provides-sinonjs-matchers"></a>
## Matchers
test should assert fuzzy.

```js
var book = {
        pages: 42,
        author: 'cjno'
      };
      var spy = test.spy();
      spy(book);
      test.sinon.assert.calledWith(spy, test.sinon.match({
        author: 'cjno'
      }));
      test.sinon.assert.calledWith(spy, test.sinon.match.has('pages', 42));
```

test should stub method differently based on argument types.

```js
var callback = test.stub();
callback.withArgs(test.sinon.match.string).returns(true);
callback.withArgs(test.sinon.match.number).throws('TypeError');
test.bool(callback('abc')).isTrue(); // Returns true
test.exception(function() {
  callback(123); // Throws TypeError
})
  .isValid(function(err) {
    if (err.name === 'TypeError') {
      return true;
    }
  });
```

Combining matchers.

```js
var stringOrNumber = test.sinon.match.string
        .or(test.sinon.match.number);
      var bookWithPages = test.sinon.match.object
        .and(test.sinon.match.has('pages'));
      var book = {
        pages: 42,
        author: 'cjno'
      };
      var spy = test.spy();
      var otherSpy = test.spy();
      spy(book);
      otherSpy(10);
      test.sinon.assert.calledWith(spy, bookWithPages);
      test.sinon.assert.calledWith(otherSpy, stringOrNumber);
```

Custom matchers.

```js
var equal10 = test.sinon.match(function(value) {
        return value === 10;
      }, 'value is not equal to 10');
      var spy = test.spy();
      var otherSpy = test.spy();
      spy(10);
      otherSpy(42);
      // ok because the argument value 10 is identical to 10 expected
      test.sinon.assert.calledWith(spy, equal10);
      test.exception(function() {
        // throws an exception because the argument value 42
        // is not identical to 10 expected
        test.sinon.assert.calledWith(otherSpy, equal10);
      })
        .hasMessage(/value is not equal to 10/);
```

<a name="unitjs-provides-sinonjs-sandbox"></a>
## Sandbox
test using test.sinon.test sandbox.

```js
return callSandboxedFn(this, slice.call(arguments), callback, handleFn);
```

