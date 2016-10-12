ublic class Observable<T> {

    final OnSubscribe<T> onSubscribe;

    /**
     * Creates an Observable with a Function to execute when it is subscribed to.
     * <p>
     * <em>Note:</em> Use {@link #create(OnSubscribe)} to create an Observable, instead of this constructor,
     * unless you specifically have a need for inheritance.
     *
     * @param f
     *            {@link OnSubscribe} to be executed when {@link #subscribe(Subscriber)} is called
     */
    protected Observable(OnSubscribe<T> f) {
        this.onSubscribe = f;
    }

    /**
     * <strong>This method requires advanced knowledge about building operators and data sources; please consider
     * other standard methods first;</strong>
     * Returns an Observable that will execute the specified function when a {@link Subscriber} subscribes to
     * it.
     * <p>
     * <img width="640" height="200" src="https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/create.png" alt="">
     * <p>
     * Write the function you pass to {@code create} so that it behaves as an Observable: It should invoke the
     * Subscriber's {@link Subscriber#onNext onNext}, {@link Subscriber#onError onError}, and
     * {@link Subscriber#onCompleted onCompleted} methods appropriately.
     * <p>
     * A well-formed Observable must invoke either the Subscriber's {@code onCompleted} method exactly once or
     * its {@code onError} method exactly once.
     * <p>
     * See <a href="http://go.microsoft.com/fwlink/?LinkID=205219">Rx Design Guidelines (PDF)</a> for detailed
     * information.
     * <dl>
     *  <dt><b>Backpressure:</b></dt>
     *  <dd>The {@code OnSubscribe} instance provided is responsible to be backpressure-aware or
     *  document the fact that the consumer of the returned {@code Observable} has to apply one of
     *  the {@code onBackpressureXXX} operators.</dd>
     *  <dt><b>Scheduler:</b></dt>
     *  <dd>{@code create} does not operate by default on a particular {@link Scheduler}.</dd>
     * </dl>
     *
     * @param <T>
     *            the type of the items that this Observable emits
     * @param f
     *            a function that accepts an {@code Subscriber<T>}, and invokes its {@code onNext},
     *            {@code onError}, and {@code onCompleted} methods as appropriate
     * @return an Observable that, when a {@link Subscriber} subscribes to it, will execute the specified
     *         function
     * @see <a href="http://reactivex.io/documentation/operators/create.html">ReactiveX operators documentation: Create</a>
     */
    public static <T> Observable<T> create(OnSubscribe<T> f) {
        return new Observable<T>(RxJavaHooks.onCreate(f));
    }

    /**
     * Returns an Observable that respects the back-pressure semantics. When the returned Observable is
     * subscribed to it will initiate the given {@link SyncOnSubscribe}'s life cycle for
     * generating events.
     *
     * <p><b>Note:</b> the {@code SyncOnSubscribe} provides a generic way to fulfill data by iterating
     * over a (potentially stateful) function (e.g. reading data off of a channel, a parser, ). If your
     * data comes directly from an asynchronous/potentially concurrent source then consider using the
     * {@link Observable#create(AsyncOnSubscribe) asynchronous overload}.
     *
     * <p>
     * <img width="640" height="200" src="https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/create-sync.png" alt="">
     * <p>
     * See <a href="http://go.microsoft.com/fwlink/?LinkID=205219">Rx Design Guidelines (PDF)</a> for detailed
     * information.
     * <dl>
     *  <dt><b>Backpressure:</b></dt>
     *  <dd>The operator honors backpressure and generates values on-demand (when requested).</dd>
     *  <dt><b>Scheduler:</b></dt>
     *  <dd>{@code create} does not operate by default on a particular {@link Scheduler}.</dd>
     * </dl>
     *
     * @param <T>
     *            the type of the items that this Observable emits
     * @param <S> the state type
     * @param syncOnSubscribe
     *            an implementation of {@link SyncOnSubscribe}. There are many static creation methods
     *            on the class for convenience.
     * @return an Observable that, when a {@link Subscriber} subscribes to it, will execute the specified
     *         function
     * @see SyncOnSubscribe#createSingleState(Func0, Action2)
     * @see SyncOnSubscribe#createSingleState(Func0, Action2, Action1)
     * @see SyncOnSubscribe#createStateful(Func0, Func2)
     * @see SyncOnSubscribe#createStateful(Func0, Func2, Action1)
     * @see SyncOnSubscribe#createStateless(Action1)
     * @see SyncOnSubscribe#createStateless(Action1, Action0)
     * @see <a href="http://reactivex.io/documentation/operators/create.html">ReactiveX operators documentation: Create</a>
     * @since 1.2
     */
    public static <S, T> Observable<T> create(SyncOnSubscribe<S, T> syncOnSubscribe) {
        return create((OnSubscribe<T>)syncOnSubscribe);
    }

    /**
     * Returns an Observable that respects the back-pressure semantics. When the returned Observable is
     * subscribed to it will initiate the given {@link AsyncOnSubscribe}'s life cycle for
     * generating events.
     *
     * <p><b>Note:</b> the {@code AsyncOnSubscribe} is useful for observable sources of data that are
     * necessarily asynchronous (RPC, external services, etc). Typically most use cases can be solved
     * with the {@link Observable#create(SyncOnSubscribe) synchronous overload}.
     *
     * <p>
     * <img width="640" height="200" src="https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/create-async.png" alt="">
     * <p>
     * See <a href="http://go.microsoft.com/fwlink/?LinkID=205219">Rx Design Guidelines (PDF)</a> for detailed
     * information.
     * <dl>
     *  <dt><b>Backpressure:</b></dt>
     *  <dd>The operator honors backpressure and generates values on-demand (when requested).</dd>
     *  <dt><b>Scheduler:</b></dt>
     *  <dd>{@code create} does not operate by default on a particular {@link Scheduler}.</dd>
     * </dl>
     *
     * @param <T>
     *            the type of the items that this Observable emits
     * @param <S> the state type
     * @param asyncOnSubscribe
     *            an implementation of {@link AsyncOnSubscribe}. There are many static creation methods
     *            on the class for convenience.
     * @return an Observable that, when a {@link Subscriber} subscribes to it, will execute the specified
     *         function
     * @see AsyncOnSubscribe#createSingleState(Func0, Action3)
     * @see AsyncOnSubscribe#createSingleState(Func0, Action3, Action1)
     * @see AsyncOnSubscribe#createStateful(Func0, Func3)
     * @see AsyncOnSubscribe#createStateful(Func0, Func3, Action1)
     * @see AsyncOnSubscribe#createStateless(Action2)
     * @see AsyncOnSubscribe#createStateless(Action2, Action0)
     * @see <a href="http://reactivex.io/documentation/operators/create.html">ReactiveX operators documentation: Create</a>
     * @since (if this graduates from Experimental/Beta to supported, replace this parenthetical with the release number)
     */
    @Experimental
    public static <S, T> Observable<T> create(AsyncOnSubscribe<S, T> asyncOnSubscribe) {
        return create((OnSubscribe<T>)asyncOnSubscribe);
    }

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

    /**
     * Transforms a OnSubscribe.call() into an Observable.subscribe() call.
     * <p>Note: has to be in Observable because it calls the package-private subscribe() method
     * @param <T> the value type
     */
    static final class OnSubscribeExtend<T> implements OnSubscribe<T> {
        final Observable<T> parent;
        OnSubscribeExtend(Observable<T> parent) {
            this.parent = parent;
        }
        @Override
        public void call(Subscriber<? super T> subscriber) {
            subscriber.add(subscribe(subscriber, parent));
        }
    }

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

    /**
     * Transform an Observable by applying a particular Transformer function to it.
     * <p>
     * This method operates on the Observable itself whereas {@link #lift} operates on the Observable's
     * Subscribers or Observers.
     * <p>
     * If the operator you are creating is designed to act on the individual items emitted by a source
     * Observable, use {@link #lift}. If your operator is designed to transform the source Observable as a whole
     * (for instance, by applying a particular set of existing RxJava operators to it) use {@code compose}.
     * <dl>
     *  <dt><b>Backpressure:</b></dt>
     *  <dd>The operator itself doesn't interfere with the backpressure behavior which only depends
     *  on what kind of {@code Observable} the transformer returns.</dd>
     *  <dt><b>Scheduler:</b></dt>
     *  <dd>{@code compose} does not operate by default on a particular {@link Scheduler}.</dd>
     * </dl>
     *
     * @param <R> the value type of the output Observable
     * @param transformer implements the function that transforms the source Observable
     * @return the source Observable, transformed by the transformer function
     * @see <a href="https://github.com/ReactiveX/RxJava/wiki/Implementing-Your-Own-Operators">RxJava wiki: Implementing Your Own Operators</a>
     */
    @SuppressWarnings("unchecked")
    public <R> Observable<R> compose(Transformer<? super T, ? extends R> transformer) {
        return ((Transformer<T, R>) transformer).call(this);
    }

    /**
     * Function that receives the current Observable and should return another
     * Observable, possibly with given element type, in exchange that will be
     * subscribed to by the downstream operators and subscribers.
     * <p>
     * This convenience interface has been introduced to work around the variance declaration
     * problems of type arguments.
     *
     * @param <T> the input Observable's value type
     * @param <R> the output Observable's value type
     */
    public interface Transformer<T, R> extends Func1<Observable<T>, Observable<R>> {
        // cover for generics insanity
    }

    /**
     * Calls the specified converter function during assembly time and returns its resulting value.
     * <p>
     * This allows fluent conversion to any other type.
     * @param <R> the resulting object type
     * @param converter the function that receives the current Observable instance and returns a value
     * @return the value returned by the function
     */
    @Experimental
    public final <R> R to(Func1<? super Observable<T>, R> converter) {
        return converter.call(this);
    }