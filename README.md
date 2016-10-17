#RxJava 說明文件 心得

不得不說RxJava的文件真的很多，多到令人覺得無從著手，當然從拋物線大大的文章是可以做基本入門，但若想要進一步了解，似乎還是從官方文件上手比較好。

官方文件-hello world! 程式碼

```java
public static void hello(String... names) {
    Observable.from(names).subscribe(new Action1<String>() {

        @Override
        public void call(String s) {
            System.out.println("Hello " + s + "!");
        }

    });
}
```

但在RxJava程式碼內的脈胳有些難追蹤，subscribe()是Observerable內的方法，它有多種參數的形式，**返回值主要是Subscription介面類的**  
由下表可看出，為什麼Action1可以、Observer也可以、Subscriber也可以 是參數了。  
**subscribeOn是線程切換，相對於observerOn**

| Modifier and Type        | Method and Description          |
| ------------- |:-------------|
|Subscription 	|subscribe()
||Subscribes to an Observable and ignores onNext and onCompleted emissions.
|Subscription 	|subscribe(Action1<? super T> onNext)
||Subscribes to an Observable and provides a callback to handle the items it emits.
|Subscription 	|subscribe(Action1<? super T> onNext, Action1<java.lang.Throwable> onError)
||Subscribes to an Observable and provides callbacks to handle the items it emits and any error notification it issues.
|Subscription 	|subscribe(Action1<? super T> onNext, Action1<java.lang.Throwable> onError, Action0 onCompleted)
||Subscribes to an Observable and provides callbacks to handle the items it emits and any error or completion notification it issues.
|Subscription 	|subscribe(Observer<? super T> observer)
||Subscribes to an Observable and provides an Observer that implements functions to handle the items the Observable emits and any error or completion notification it issues.
|Subscription 	|subscribe(Subscriber<? super T> subscriber)
||Subscribes to an Observable and provides a Subscriber that implements functions to handle the items the Observable emits and any error ||or completion notification it issues.
|Observable<T> 	|subscribeOn(Scheduler scheduler)
||Asynchronously subscribes Observers to this Observable on the specified Scheduler.

在Observerable中OnSubscribe<T>是內部介面類，且繼承Action1<Subscriber<? super T>>

```java
    /**
     * Invoked when Observable.subscribe is called.
     * @param <T> the output value type
     */
    public interface OnSubscribe<T> extends Action1<Subscriber<? super T>> {
        // cover for generics insanity
    }

    /**
     * Operator function for lifting into an Observable.
     * @param <T> the upstream's value type (input)
     * @param <R> the downstream's value type (output)
     */
    public interface Operator<R, T> extends Func1<Subscriber<? super R>, Subscriber<? super T>> {
        // cover for generics insanity
    }    
```

另外也要注意執行的線程是否可用，若無線程會出錯。

Observable的點方法主要可分為二大類，一是上述的OnSubscribe，另一類是Operator extends Func1

```java
public interface Action1<T> extends Action {
    void call(T t);
}

public interface Func1<T, R> extends Function {
    R call(T t);
}
```
最主要的差異是Action1是void call(T t)，而Func1是R call(T t)，一個無返回類，另一個有返回類。

`Observable.subscribe(Subscriber)` 的內部實現是這樣的（僅核心代碼）：

```java
// 注意：這不是 subscribe() 的源碼，而是將源碼中與性能、兼容性、擴展性有關的代碼剔除後的核心代碼。
// 如果需要看源碼，可以去 RxJava 的 GitHub 倉庫下載。
public Subscription subscribe(Subscriber subscriber) {
    subscriber.onStart();
    onSubscribe.call(subscriber);
    return subscriber;
}
```
可以看到，`subscriber()` 做了3件事：

1. 調用 `Subscriber.onStart()` 。這個方法在前面已經介紹過，是一個可選的準備方法。
2. 調用 `Observable` 中的 `OnSubscribe.call(Subscriber)` 。在這裡，事件發送的邏輯開始運行。從這也可以看出，在 `RxJava` 中， `Observable` 並不是在創建的時候就立即開始發送事件，而是在它被訂閱的時候，即當 `subscribe()` 方法執行的時候。
3. 將傳入的 `Subscriber` 作為 `Subscription` 返回。這是為了方便 `unsubscribe()`.


```java
    /**
     * <strong>This method requires advanced knowledge about building operators; please consider
     * other standard composition methods first;</strong>
     * Lifts a function to the current Observable and returns a new Observable that when subscribed to will pass
     * the values of the current Observable through the Operator function.
     * <p>
     * In other words, this allows chaining Observers together on an Observable for acting on the values within
     * the Observable.
     * <p> {@code
     * observable.map(...).filter(...).take(5).lift(new OperatorA()).lift(new OperatorB(...)).subscribe()
     * }
     * <p>
     * If the operator you are creating is designed to act on the individual items emitted by a source
     * Observable, use {@code lift}. If your operator is designed to transform the source Observable as a whole
     * (for instance, by applying a particular set of existing RxJava operators to it) use {@link #compose}.
     * <dl>
     *  <dt><b>Backpressure:</b></dt>
     *  <dd>The {@code Operator} instance provided is responsible to be backpressure-aware or
     *  document the fact that the consumer of the returned {@code Observable} has to apply one of
     *  the {@code onBackpressureXXX} operators.</dd>
     *  <dt><b>Scheduler:</b></dt>
     *  <dd>{@code lift} does not operate by default on a particular {@link Scheduler}.</dd>
     * </dl>
     *
     * @param <R> the output value type
     * @param operator the Operator that implements the Observable-operating function to be applied to the source
     *             Observable
     * @return an Observable that is the result of applying the lifted Operator to the source Observable
     * @see <a href="https://github.com/ReactiveX/RxJava/wiki/Implementing-Your-Own-Operators">RxJava wiki: Implementing Your Own Operators</a>
     */
    public final <R> Observable<R> lift(final Operator<? extends R, ? super T> operator) {
        return create(new OnSubscribeLift<T, R>(onSubscribe, operator));
    }
```
原始碼上，只有到new OnSubscribeLift<..>(..)。扔物線上的多了一些程式碼

##2) 變換的原理：lift()

這些變換雖然功能各有不同，但實質上都是針對事件序列的處理和再發送。而在 RxJava 的內部，它們是基於同一個基礎的變換方法： lift(Operator)。首先看一下 lift() 的內部實現（僅核心代碼）：

```java
// 注意：這不是 lift() 的源碼，而是將源碼中與性能、兼容性、擴展性有關的代碼剔除後的核心代碼。
// 如果需要看源碼，可以去 RxJava 的 GitHub 倉庫下載。
public <R> Observable<R> lift(Operator<? extends R, ? super T> operator) {
    return Observable.create(new OnSubscribe<R>() {
        @Override
        public void call(Subscriber subscriber) {
            Subscriber newSubscriber = operator.call(subscriber);
            newSubscriber.onStart();
            onSubscribe.call(newSubscriber);
        }
    });
}
```

這段代碼很有意思：它生成了一個新的 Observable 並返回，而且創建新 Observable 所用的參數 OnSubscribe 的回調方法 call() 中的實現竟然看起來和前面講過的 Observable.subscribe() 一樣！然而它們並不一樣喲~不一樣的地方關鍵就在於第二行 onSubscribe.call(subscriber) 中的 onSubscribe **所指代的物件不同**（高能預警：接下來的幾句話可能會導致身體的嚴重不適）——

subscribe() 中這句話的 onSubscribe 指的是 Observable 中的 onSubscribe 物件，這個沒有問題，但是 lift() 之後的情況就複雜了點。
當含有 lift() 時：

1. lift() 創建了一個 Observable 後，加上之前的原始 Observable，已經有兩個 Observable 了；
2. 而同樣地，新 Observable 裡的新 OnSubscribe 加上之前的原始 Observable 中的原始 OnSubscribe，也就有了兩個 OnSubscribe；
3. 當用戶調用經過 lift() 後的 Observable 的 subscribe() 的時候，使用的是 lift() 所返回的新的 Observable ，於是它所觸發的 onSubscribe.call(subscriber)，也是用的新 Observable 中的新 OnSubscribe，即在 lift() 中生成的那個 OnSubscribe；
4. 而這個新 OnSubscribe 的 call() 方法中的 onSubscribe ，就是指的原始 Observable 中的原始 OnSubscribe ，在這個 call() 方法裡，新 OnSubscribe 利用 operator.call(subscriber) 生成了一個新的 Subscriber（Operator 就是在這裡，通過自己的 call() 方法將新 Subscriber 和原始 Subscriber 進行關聯，並插入自己的『變換』代碼以實現變換），然後利用這個新 Subscriber 向原始 Observable 進行訂閱。
這樣就實現了 lift() 過程，有點像一種代理(delegate)機制，通過事件攔截和處理實現事件序列的變換。

精簡掉細節的話，也可以這麼說：在 Observable 執行了 lift(Operator) 方法之後，會返回一個新的 Observable，這個新的 Observable 會像一個代理一樣，負責接收原始的 Observable 發出的事件，並在處理後發送給 Subscriber。

Observable內部的泛型方法lift，返回值類是`Observable<R>`，參數是(`final Operator<? extends R, ? super T> operator`) ，主要用在變換Operator。

