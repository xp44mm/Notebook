# rxjs后缀

## with

startWith,endWith:取发射的内容，参数同from

timeoutWith

## while

skipWhile,takeWhile取谓词函数，参数同filter

## last

takeLast,skipLast,与不带last后缀的函数相仿，都取一个整数表示数量，只是从后向前计数。

last最后一个符合条件的元素，参数同filter

publishLast,最后一个通知，就是最后一个，没有参数。

### Map,MapTo后缀

- [ concatMap](https://rxjs-dev.firebaseapp.com/api/operators/concatMap)
- [ concatMapTo](https://rxjs-dev.firebaseapp.com/api/operators/concatMapTo)
- [ exhaustMap](https://rxjs-dev.firebaseapp.com/api/operators/exhaustMap)
- [ ~~flatMap~~](https://rxjs-dev.firebaseapp.com/api/operators/flatMap)
- [ mergeMap](https://rxjs-dev.firebaseapp.com/api/operators/mergeMap)
- [ mergeMapTo](https://rxjs-dev.firebaseapp.com/api/operators/mergeMapTo)
- [ switchMap](https://rxjs-dev.firebaseapp.com/api/operators/switchMap)
- [ switchMapTo](https://rxjs-dev.firebaseapp.com/api/operators/switchMapTo)

先执行map或者mapTo管道操作, 这两个操作都返回高阶可观察，再执行相应行为的All管道操作。

### All后缀

- [ combineAll](https://rxjs-dev.firebaseapp.com/api/operators/combineAll)
- [ concatAll](https://rxjs-dev.firebaseapp.com/api/operators/concatAll)
- [ mergeAll](https://rxjs-dev.firebaseapp.com/api/operators/mergeAll)
- [ switchAll](https://rxjs-dev.firebaseapp.com/api/operators/switchAll)
- [ zipAll](https://rxjs-dev.firebaseapp.com/api/operators/zipAll)

与对应的创造方法相同，只是创造方法是组合两个可观察，而All相当于使用对应的创造方法对可观察的可观察应用了一次reduce

### toggle后缀

- [ bufferToggle](https://rxjs-dev.firebaseapp.com/api/operators/bufferToggle)
- [ windowToggle](https://rxjs-dev.firebaseapp.com/api/operators/windowToggle)

openings发射打开通知，closing根据opening发射的值决定何时关闭。是when后缀的加强版。

### when后缀

- [ bufferWhen](https://rxjs-dev.firebaseapp.com/api/operators/bufferWhen)
- [ windowWhen](https://rxjs-dev.firebaseapp.com/api/operators/windowWhen)
- [ delayWhen](https://rxjs-dev.firebaseapp.com/api/operators/delayWhen)
- [ repeatWhen](https://rxjs-dev.firebaseapp.com/api/operators/repeatWhen)
- [ retryWhen](https://rxjs-dev.firebaseapp.com/api/operators/retryWhen)

表示对连接结合部精确控制，参数是返回可观察的函数。

### count后缀

- [ bufferCount](https://rxjs-dev.firebaseapp.com/api/operators/bufferCount)
- [ windowCount](https://rxjs-dev.firebaseapp.com/api/operators/windowCount)

下面这个count后缀只是巧合。
- [ refCount](https://rxjs-dev.firebaseapp.com/api/operators/refCount)

没有count后缀，参数是count的有take,skip,repeat,retry,

until后缀

skipUnitl, takeUntil

参数是一个发射改变信号的可观察。

time后缀

- [ auditTime](https://rxjs-dev.firebaseapp.com/api/operators/auditTime)
- [ bufferTime](https://rxjs-dev.firebaseapp.com/api/operators/bufferTime)
- [ debounceTime](https://rxjs-dev.firebaseapp.com/api/operators/debounceTime)
- [ sampleTime](https://rxjs-dev.firebaseapp.com/api/operators/sampleTime)
- [ throttleTime](https://rxjs-dev.firebaseapp.com/api/operators/throttleTime)
- [ windowTime](https://rxjs-dev.firebaseapp.com/api/operators/windowTime)

参数是毫秒数