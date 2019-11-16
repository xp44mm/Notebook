Chapter 9. Toward testable, reactive programs

9.1. Testing is inherently built into functional programs

新建`src/average.test.js`文件：

```javascript
function average(arr) {
    let len = arr.length;
    total = arr.reduce((a, b) => a + b);//error
    return Math.floor(total / len);
}

describe('Average numbers', function () {
    test('Leak the variable total', function () {
        expect(average([80, 90, 100])).toBe(90);
    });
});
```

执行命令：

```javascript
npm run j -- src/ave
```

这里的参数表示路径，提示信息会有：

```javascript
Ran all test suites matching /src\\ave/i.
```

表示路径不区分大小写。使用正则表达式

被测试的函数内`total`丢失关键字`let`,测试给出错误：

```
ReferenceError: total is not defined
```

修改后测试会通过。

新建`src/notEmpty.test.js`文件：

```javascript
const notEmpty = input => !!input && input.trim().length > 0;

describe('Validation', () => {
    test('Should validate that a string is not empty', () => {
        expect(notEmpty('some input')).toBe(true);
        expect(notEmpty(' ')         ).toBe(false);
        expect(notEmpty(null)        ).toBe(false);
        expect(notEmpty(undefined)   ).toBe(false);
    });
});
```

9.2. Testing asynchronous code and promises

9.2.1. Testing AJAX requests

`ajax.js`文件：

```javascript
import { XMLHttpRequest } from 'xmlhttprequest'

export const ajax = (url, success, error) => {
    let req = new XMLHttpRequest();
    req.responseType = 'json';
    req.open('GET', url);
    req.onload = function () {
        if (req.status == 200) {
            let data = JSON.parse(req.responseText);
            success(data);
        }
        else {
            req.onerror();
        }
    }
    req.onerror = function () {
        if (error) {
            error(new Error('IO Error'));
        }
    };
    req.send();
};
```

因为node.js不支持`XMLHttpRequest`类，需要安装包`'xmlhttprequest'`

为了测试ajax，添加服务器端控制器代码`ValuesController.cs`文件：

```C#
[Route("api/[controller]")]
public class ValuesController : Controller
{
    private static string[] testData = new string[] {
        "github.com/Reactive-Extensions/RxJS",
        "github.com/ReactiveX/RxJS",
        "xgrommx.github.io/rx-book",
        "reactivex.io",
        "egghead.io/technologies/rx",
        "rxmarbles.com",
        "https://www.manning.com/books/rxjs-in-action"
    };

    // GET api/<controller>/rx
    [HttpGet("{query}")]
    public string[] Get(string query)
    {
        if (query.Length > 0)
        {
            return (from item in ValuesController.testData
                    where item.StartsWith(query)
                    select item
                    ).ToArray();
        }
        else
        {
            return new string[] { };
        }
    }
}
```

测试`ajax.test.js`文件：

```javascript
import { ajax } from './ajax'

describe('Ajax test', () => {
    test('Should fetch Wikipedia pages for search term "r"', done => {
        const searchTerm = 'r';
        const url = `http://localhost:53849/api/values/${searchTerm}`;

        const success = results => {
            expect(results).toBeInstanceOf(Array)
            done();
        };

        const error = (err) => {
            done(err);
        };

        ajax(url, success, error);
    });

    test('Should fail for invalid URL', done => {
        const url = 'invalid-url';

        const success = data => {
            done(new Error('Should not have been successful!'));
        };

        const error = (err) => {
            expect(err).toEqual(new Error('IO Error'))
            done();
        };

        ajax(url, success, error);
    });

});
```

启动服务器端的服务，然后执行测试。

这是异步测试需要在回调方法的末尾调用输入的函数`done`，其可以带任意个参数。

9.2.2. Working with Promises

新建`promise.js`文件：

```javascript
import { XMLHttpRequest } from 'xmlhttprequest'

export const ajax = url => new Promise((resolve, reject) => {
    let req = new XMLHttpRequest();
    req.responseType = 'json';
    req.open('GET', url);
    req.onload = () => {
        if (req.status == 200) {
            let data = JSON.parse(req.responseText);
            resolve(data);
        }
        else {
            reject(new Error(req.statusText));
        }
    };
    req.onerror = () => {
        reject(new Error('IO Error'));
    };
    req.send();
});
```

新建`promise.test.js`测试文件：

```javascript
import { from } from 'rxjs';
import { ajax } from './promise'

describe('Ajax with promises', () => {
    test('Should fetch Wikipedia pages for search term "rx"',
        () => {
            expect.assertions(1);
            const searchTerm = 'rx';
            const url = `http://localhost:53849/api/values/${searchTerm}`;

            return expect(ajax(url))
                .resolves.toEqual(['rxmarbles.com'])
        });
});
```

启动服务器端的服务，然后执行测试。

9.3 Testing reactive streams

Listing 9.5 Testing a stream that adds up all numbers of an array

新建`adds.test.js`测试文件：

```javascript
import { from } from 'rxjs';
import { take, reduce, delay } from 'rxjs/operators';

const adder = (total, delta) => total + delta;

describe('Adding numbers', function () {
    test('Should add numbers together', function () {
        from([1, 2, 3, 4, 5, 6, 7, 8, 9]).pipe(
            reduce(adder)
        )
            .subscribe(total => {
                expect(total).toBe(45);
            });
    });

    test('Should add numbers from a generator', function () {
        function* numbers() {
            let start = 0;
            while (true) {
                yield start++;
            }
        }

        from(numbers()).pipe(
            take(10),
            reduce(adder)
        )
            .subscribe(total => {
                expect(total).toBe(45);
            });
    });

    test('Should add numbers together with delay', function (done) {
        from([1, 2, 3, 4, 5, 6, 7, 8, 9]).pipe(
            reduce((total, delta) => total + delta),
            delay(1000),
        )
            .subscribe(total => {
                expect(total).toBe(45);
            }, null, () => {
                console.log('delay completed!')
                done()
            });
    });
});
```

代码包括三组测试，分别演示了同步流、同步生成器函数、异步流的测试。

演示rxjs自带ajax函数的用法：

新建`fetchData.js`文件：

```javascript
import { ajax } from 'rxjs/ajax';
import { map } from 'rxjs/operators';

export function fetchData(searchTerm) {
    const url = `http://localhost:53849/api/values/${searchTerm}`;
    return ajax(url)
    .pipe(map(resp => resp.response))
}
```

新建`fetchData.test.js`测试文件：

```javascript
import { fetchData } from './fetchData'
import { XMLHttpRequest } from 'xmlhttprequest'

global.XMLHttpRequest || (global.XMLHttpRequest = XMLHttpRequest)

describe('Ajax test', () => {
    test('the data is peanut butter', done => {
        fetchData('rx').subscribe(results => {
            expect(results).toEqual(["rxmarbles.com"])
            done();
        })
    });
});
```

测试是运行在node.js环境下的，它没有`XMLHttpRequest`功能，所以测试前需要配齐这个功能。本文件的第一句就是做这个的。

9.4 Making streams testable

新建`runInterval.js`文件：

```javascript
import { filter, map, reduce, take } from 'rxjs/operators';

const isEven = num => num % 2 === 0;
const square = num => num * num;
const add = (a, b) => a + b;

export const runInterval = (source$) =>
    source$.pipe(
        take(10),
        filter(isEven),
        map(square),
        reduce(add),
    )
```

可测试的代码应该被分成来源、管道、订阅。

新建`runInterval.test.js`测试文件：

```javascript
import { interval } from 'rxjs';
import { runInterval } from './runInterval';

test('Should square and add even numbers', function (done) {
    jest.setTimeout(20000);
    runInterval(interval(1000))
        .subscribe({
            next: total => expect(total).toBe(120),
            error: err => expect(err).toThrow(),
            complete: done
        });
});
```

这个测试文件的执行需要耗时等待12s之多。所以将超时设置为20s，以避免测试失败。

9.5. Scheduling values in RxJS

新建`scheduler.test.js`测试文件：

```javascript
import { asyncScheduler, range } from 'rxjs';
import { observeOn, tap } from 'rxjs/operators';
import { TestScheduler } from 'rxjs/testing';

describe('Test Scheduler', function () {
    test('Should schedule things in order', function () {
        let stored = [];

        let store = state => () => stored.push(state);

        let scheduler = new TestScheduler(null);

        scheduler.schedule(store(1));
        scheduler.schedule(store(2));
        scheduler.schedule(store(3));
        scheduler.schedule(store(4));
        scheduler.schedule(store(5));

        scheduler.flush();

        expect(stored).toEqual([1, 2, 3, 4, 5]);
    });

    test('Emits values synchronously on default scheduler', function () {
        let temp = [];
        range(1, 5).pipe(
            tap([].push.bind(temp))
        )
            .subscribe(value => {
                expect(temp).toHaveLength(value);
                expect(temp).toContain(value);
            });
    });

    test('Emits values on an asynchronous scheduler', function (done) {
        let temp = [];
        range(1, 5, asyncScheduler)
            .pipe(
                tap([].push.bind(temp))
            ).subscribe(value => {
                expect(temp).toHaveLength(value);
                expect(temp).toContain(value);
            }, done, done);
    });

    test('Emits values on an asynchronous scheduler by observeOn', function (done) {
        let temp = [];
        range(1, 5).pipe(
            observeOn(asyncScheduler),
            tap([].push.bind(temp)),
        ).subscribe(value => {
            expect(temp).toHaveLength(value);
            expect(temp).toContain(value);
        }, done, done);
    });
});
```

9.6. Augmenting virtual reality

```javascript
test('Create time from a marble diagram', function () {
    let scheduler = new TestScheduler();
    let time = scheduler.createTime('-----|');
    expect(time).toBe(50);
});
```

9.6.1 Playing with marbles

```javascript
test('Should parse a marble string into a series of notifications',
    function () {
        let result = TestScheduler.parseMarbles(
            '--a---b---|',
            { a: 'A', b: 'B' });

        expect(result).toEqual([
            { frame: 20, notification: Notification.createNext('A') },
            { frame: 60, notification: Notification.createNext('B') },
            { frame: 100, notification: Notification.createComplete() }
        ]);
    });
```

时间的长度`frame`等于弹珠之前的字符数乘以10.

Listing 9.9 Testing the map() operator

```javascript
test('Should map multiple values', function () {
    let scheduler = new TestScheduler((actual, expected) => expect(actual).toEqual(expected));
    let source = scheduler.createColdObservable(
        '--1--2--3--4--5--6--7--8--9--|');
    let r = map(x => x * x)(source)
    let expected = '--a--b--c--d--e--f--g--h--i--|';
    scheduler.expectObservable(r).toBe(expected,
        {
            a: 1, b: 4, c: 9, d: 16, e: 25,
            f: 36, g: 49, h: 64, i: 81
        });
    scheduler.flush();
});
```

以上代码重构为v6版本：

```javascript
test('Should map multiple values v6', function () {
    let scheduler = new TestScheduler((actual, expected) => expect(actual).toEqual(expected));

    scheduler.run(({ cold, hot, expectObservable, expectSubscriptions, flush }) => {
        const inp = '--1--2--3--4--5--6--7--8--9--|';
        const exp = '--a--b--c--d--e--f--g--h--i--|';

        const result = cold(inp).pipe(
            map(x => x * x)
        );

        expectObservable(result).toBe(exp, {
            a: 1, b: 4, c: 9, d: 16, e: 25,
            f: 36, g: 49, h: 64, i: 81
        });
    });
});
```

Listing 9.10 Testing the debounceTime operator

```javascript
test('Should delay all element by the specified time', function () {
    let scheduler = new TestScheduler((actual, expected) => expect(actual).toEqual(expected));

    let source = scheduler.createHotObservable(
                   '-a--------b------c----|');
    let expected = '------a--------b------(c|)';

    let r = debounceTime(50, scheduler)(source);
    scheduler.expectObservable(r).toBe(expected);
    scheduler.flush();
});
```

以上代码重构为v6版本：

```javascript
    test('Should delay all element by the specified time v6', function () {
        let scheduler = new TestScheduler((actual, expected) => expect(actual).toEqual(expected));

        scheduler.run(({ cold, hot, expectObservable, expectSubscriptions, flush }) => {
            const inp = '-a 80ms b 60ms c 40ms |';
            const exp = '- 50ms a 80ms b 51ms (c|)';

            const result = hot(inp).pipe(
                debounceTime(50)
            );

            expectObservable(result).toBe(exp);
        });
    });

```

结果流的计算过程如下：

`a`发射50ms后，结果流收到`a`。

```javascript
'-a'.length + 50 = '-'.length + exp + 'a'.length
```
`b`发射50ms后，结果流收到`b`。简便起见，字符串省略`.length`

```javascript
'-a' + 80 + 'b' + 50 = '-' + 50 + 'a' + exp + 'b'
```

`c`发射到流完成只有40ms，则在输入流完成时，结果流收到`c`。

```javascript
'-a' + 80 + 'b' + 60 + 'c' + 40 + '|' = '-' + 50 + 'a' + 80 + 'b' + exp + 'c'
```
Listing 9.11 Speeding up runInterval() with the virtual time scheduler

```javascript
test('Should square and add even numbers', function () {
    let scheduler = new TestScheduler((actual, expected) => expect(actual).toEqual(expected));

    let source = scheduler.createColdObservable(
                   '-1-2-3-4-5-6-7-8-9-|');
    let expected = '-------------------(s|)';

    let r = runInterval(source);
    scheduler.expectObservable(r).toBe(expected, { s: 120 });
    scheduler.flush();
});
```
窍门：对齐输入与输出

重构v6版本：

```javascript
test('Should square and add even numbers v6', function () {
    let scheduler = new TestScheduler((actual, expected) => expect(actual).toEqual(expected));

    scheduler.run(({ cold, hot, expectObservable, expectSubscriptions, flush }) => {
        const inp = '-1-2-3-4-5-6-7-8-9-|';
        const exp = '-------------------(s|)';
        const result = runInterval(cold(inp))
        expectObservable(result).toBe(exp, { s: 120 });
    });
});
```

9.6.3. Refactoring your search stream for testability

重构后的search，列在`search.refac.js`文件中：

```javascript
import { fromEvent, fromPromise } from 'rxjs';
import { debounceTime, defaultIfEmpty, filter, map, pluck, switchMap, tap } from 'rxjs/operators';
import { ajax } from './promise';

const notEmpty = input => !!input && input.trim().length > 0;
export const search$ = (source$, fetchResult$, url = '', scheduler = null) =>
    source$.pipe(
        debounceTime(500, scheduler),
        filter(notEmpty),
        tap(term => console.log(`Searching with term ${term}`)),
        map(query => url + query),
        switchMap(fetchResult$),
    )
let usage = () => {
    search$(
        pluck('target', 'value')(fromEvent(inputText, 'keyup')),
        query =>
            fromPromise(ajax(query)).pipe(
                pluck('query', 'search'),
                defaultIfEmpty([]),
            ),
        'http...',
    ).subscribe(arr => {
        count.innerHTML = `${result.length} results`;
        if (arr.length === 0) {
            clearResults(results);
        }
        else {
            appendResults(results, arr);
        }
    });
}
```

测试`search.test.js`文件：

```javascript
import { of } from 'rxjs';
import { TestScheduler } from 'rxjs/testing';
import { search$ } from './search.refac';

function frames(n = 1, unit = '-') {
    return (n === 1) ? unit :
        unit + frames(n - 1, unit);
}

describe('Search component', function () {
    const results_1 = [
        'rxmarbles.com',
        'https://www.manning.com/books/rxjs-in-action'
    ];

    const results_2 = [results_1[1]]

    const searchFn = term => {
        let r = [];
        if (term.toLowerCase() === 'rx') {
            r = results_1;
        }
        else if (term.toLowerCase() === 'rxjs') {
            r = results_2;
        }
        return of(r);
    };

    test('Should test the search stream with debouncing', function () {
        let searchTerms = {
            a: 'r',
            b: 'rx',
            c: 'rxjs',
        };

        let scheduler = new TestScheduler((actual, expected) => expect(actual).toEqual(expected));
        let source = scheduler.createHotObservable(
            '-a-b-' + frames(50) + '-c|', searchTerms);
        let expected = frames(50) + '---f---(s|)';
        let r = search$(source, searchFn, '', scheduler);
        scheduler.expectObservable(r).toBe(expected,
            {
                f: results_1,
                s: results_2
            });

        scheduler.flush();
    });
});
```


