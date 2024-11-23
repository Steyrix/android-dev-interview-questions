# android-dev-interview-questions
My list of **esoteric/not very oftenly met/relatively difficult** technical questions that one can face during a technical interview for the Android Developer role.
From my experience, these questions are oftenly met at the technical interview for the Senior grade.
This list does not include many of the standard questions as they all can be easily found on the internet.

## Kotlin section

#### What are GC.roots?
    - Classes loaded via system’s ClassLoader.
    - Stack of local variables
    - Active threads (thread root)
    - JNI references
    - Synchronization’s monitors
    - JVM related specific objects

#### Sequence vs List
Sequence processes elements on demand one by one, avoiding creating of intermediate collections. 
List’s operators create intermediate collections and process elements as a whole collection. Sequence can be used to process enormous amount of elements.

#### Sorting sequence 
Sequence is converted to MutableList, get sorted and then the list is converted to sequence.

#### Inner class vs nested class
Nested class can be created via class definition of outer class.
Whereas inner class requires an instance of outer class and can access its fields.  

#### Enum vs sealed
Enum value is not truly a new type whereas sealed type subclass is truly a new different types.

#### When is it necessary to use sealed interface instead of sealed class?
Sealed interface usage allows a subtype to participate in a more complex class hierarchy and extend a class. 
Java prohibits extending multiple classes while multiple interfaces can be implemented.

#### When is it necessary to use sealed classes instead of sealed interfaces?
Sealed classes can be used to define a set of properties that has default values. Default property initialization
is not available in interfaces.

#### What are Delegates?
Can use instance of a class/interface to pass the behavior

#### What is Java Memory Model
Shortly, Java Memory model states that each thread has its own segment of memory.

#### How do Atomics work?
Atomic uses CompareAndSet / CompareAndSwap operations which is primitive CPU-level operation. 
The CAS operation compares if value at known location equals to the existing value and if it does - the old value is swapped with the new value.

#### Kotlin in/out
**out** is equivalent to <? extends T>
**in** is equivalent to <? super T>  JVM ensures type safety so we cannot assign subtype generic object to supertype generic object and later modify supertype object with illegal types.  

#### Kotlin in/out ensuring type safety
For example **in** allows us to assign any subtype type to any supertype since it can only be consumed. 
**out** restricts consumption of illegal types (add/set element of integer to the list of strings)
In restrict production of illegal types (consume String to compare it with Number)


## Coroutines section

#### How does delay() works?
delay() does not block thread allowing another work to run. 
Suspended code’s timer is observed constantly, coroutine’s queue depends on it and reorders queue based on remaining delay.

#### Why synchronized cannot be used with suspend methods?
Synchronized cannot be used with coroutines since they use continuation pattern. 
When the coroutine will reach the critical point, it will suspend and release the lock for another thread to acquire. 

#### Is there a possibility to manually create Deferred<T>?
There is a possibility to create deferred using the constructor of CompletableDeferred<T> without async coroutine builder. 
This allows to pass deferred anywhere and then subscribe to it.

#### How do multiple launch calls in a single thread arranged?
Launch gets enqueued. The queue can be reordered base on suspensions points and suspending time.

#### How to dispatch work on IO/Default and guarantee only one thread will be used?
Dispatcher.limitedParalellism() - method that is used to limit dispatcher’s thread pool.

#### How is continuation passing approach implemented while JVM does not provide tools for it?
When suspend function launched, it creates a continuation object referring to it. 
If continuation object is acquired first time the function is in **START** state.
Suspend functions return right after suspension point is met. When they should be resumed, continuation object is acquired and 
if it was already acquired by the function, function enters **RESUMED** state.
Continuation object is able to store parent function's continuation to provide structured concurrency.

## Android section

#### viewModelScope vs lifecycleScope:
viewModelScope is bound to ViewModel’s lifecycle, whereas lifecycleScope is bound to specific’s lifecycleOwner.
viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) is an example of coroutine restart in STARTED state.
flowWithLifecycle(viewLifecycleOwner.lifecycle, Lifecycle.State.STARTED) is an example of processing a flow in STARTED state;
whenStarted() is a suspension point for coroutine.

#### How to change view’s size in onMeasure() callback?
Override onMeasure() method, using parent’s dimension passed via widthMeasureSpec  and heightMeasureSpec 
Parameters. Dimensions should be changed via setMeasuredDimension(). Otherwise view’s size will be 100x100.

#### What is purpose of implicit Intents?
Implicit intents does not target specific component but describe the action and their parameters later «catched» by intent-filters of other components (e.g. other application’s BroadcastReceiver). If multiple intent-filters can process implicit intent, dialog with choice of application displays.

#### Broadcast receiver, case when onReceive() performs a long running operation
By default, onReceive() works on main thread. Long task then can result in ANR.
Long running operations should be performed via Service. For that goAsync() should be called in onReceive() and BroadcastReceiver.PendingResult should be passed to the background thread.
Alternative is passing the job to JobScheduler

#### What are WEBp format’s benefits?
WEBp approximately smaller than other images by 30% and is faster to download.

#### Sp vs dp?
Scalable pixels vs density-independent pixels. SP preservers user’s font settings

#### What is purpose of another process?
To launch third party API in another process and prevent it interfering with main process stack OR perform application components-related work outside.

#### What are FragmentManager’s commit operation types?
- commitNow - immediately perform transaction
- commitAllowStateLoss - perform commit even after onRestoreInstanceState.
- commit - regular commit operation that is enqued as the task on UI thread.

#### What are Parcelable limitations>?
Parcelable memory limitation is 1MB by default. 

## Compose section

#### How does compose redraws only those views that are changed?
Compose is representing composable as a node tree. When the state of node changes a recomposition is triggered. After that all the child nodes get updated.

#### What is Mutable/Immutable/Stable in Compose?
If composable’s parameters are only immutable ones, it can be marked as skippable.
Data classes with only immutable values and primitives are Immutable by default, whereas all plain collections are mutable. Stable types are types, which can change during the work, however they «promise» to notify Compose that the changes are made. The one way to do this is to use MutableState (compose «subscribes» to mutable states)

#### What is MutableState?
MutableState is a value holder to which current RecomposeScope will subscribe. If the value is written to or changed, all subscribed instances of RecomposeScope will schedule recompositions.  

#### What is remember?
Remember allows to remember the value from the previous compose invocation.  How do Compose works under the hood?
The calling context of Composition is called «composer» that acts as the parameter that we send to every composable function. Composer containing Gap Buffer which acts as an array where empty space is allocated «in case» composable function will be updated and new composable will be attached to it. It is only needed for restartable Composables. The gap can be moved from functions to function, this operation is O(n), however we should build the composition in a way ensuring that this operation will not happen very often.  When composable is called, composer.start() gets executed. It inserts an object with integer id to the buffer, remember expressions are also inserting object to the buffer. The values of mutableStates will also be stored in buffer. Composer is passed to every inner composable and will insert new Group object, values after calling «start» with now different id.  What are Composition Group objects?
Group objects are inserted to the graph if the composables referring to them can change UI. When changes are made, composer.start is called with different group id and compiler detects that it doesn’t match previous group id, so the cursor is moved to the currently activated group and empty space allocated after it.

#### What is CompositionLocalProvider?
Composition local is part of the UI three that contains values for all the children composables to use.
We can implicitly provide values using CompositionLocalProvider without needing to pass them explicitly (via parameters).
Can be either static or dynamic. By default they are dynamic 

#### How are animations performed without recomposition?
One can override render phase of composition, without triggering recompose and layout/measure phases.

## DI section

#### What is ServiceLocator?
ServiceLocator is an alternative to DI where all the dependencies are obtained from application-scoped global object.

#### Dagger vs Koin
Dagger use annotations and generates code like factories and etc. Whereas Koin is based on Service Locator pattern and accesses registry of all factories, where key is the full name of the class. These factories afterwards use to create needed dependencies and pass them to destination point. Dagger can detect issues during compile time whereas Koin can cause runtime exception.

#### What are Dagger Subcomponents?
Subcomponents are used to extend and inherit parent graph. They are also used for encapsulation and are referred to scopes.  Subcomponents can be included and modules and can include modules. Modules which are included in subcomponent are used to provide implementation that are later provided without details to modules which include subcomponents.
