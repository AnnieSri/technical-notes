# 10 Exceptions

## Item 69
- Use exceptions only for exceptional conditions
- we should not use them for ordinary conditional flow like this:
```Java
try {
    int i = 0;
    while (true) {
        range[i++].climb();
    }
} catch (ArrayIndexOutOfBoundsException e) {}
```
- this approach of using exceptions is wrong on different levels:
  - it is confusing as it's not the intention of the exceptions
  - JVM will not optimize these (compared to when we use Arrays.length() for instance)
  - we may catch an actual exception here unintentionally without realizing it
- a well designed API must not force its clients to use exceptions for ordinary workflow
  - e.g. Iterator has `hasNext()` which you can use before calling `next()`, so you don't need try/catch here
  - in general, when a method is state-dependant, we should have a state-testing method to call before calling our method to make sure we can safely call it.

## Item 70
- Use checked exceptions for recoverable conditions and runtime exceptions for programming errors
- Java provides three kinds of throwables:
  1. *checked exceptions*
    - when we declare a method throws an exception
    - the caller needs to catch it and resolve it or propagate it upwards
    - they are meant to be used in conditions from which the caller can reasonably be expected to recover
    - it is a bad idea to just catch these exceptions and ignore them.
    - provide methods on checked exceptions to aid in recovery. e.g. we have a payment w/ credit card method that throws exception if funds are not sufficient. We can provide an accessor method to query the amount of shortfall. This enables the caller to relay the amount to the shopper
  2. *runtime exceptions*
    - these exceptions are usually result of precondition violation which is client failure to adhere to the contract established by the API specifications
    - they are result of programming errors
    - e.g. the contract for array access specifies the indices to be between 0 and array's length minus 1, using something outside this range will throw `ArrayIndexOutOfBoundsException`
  3. *errors*
    - there is a strong convention that errors are reserved for use by the JVM
    - such as `OutOfMemoryError`, `ThreadDeath`, etc.
    - all the unchecked exceptions you implement should subclass `RuntimeException`

## Item 71
- Avoid unnecessary use of checked exceptions
- when used sparingly they increase reliability of programs, when overused, they make APIs painful to use
- we need to use them when we believe user can recover the situation when getting the exception
  - if they just log the stack trace or ignore it (as in like they can do nothing else) it doesn't make sense to make it a checked exception. This just makes it harder to use as you have to use a try/catch or propagate it outward!

## Item 72
- Favor the use of standard exceptions
- Java libraries provide a set of exceptions that covers most of exception throwing needs of most APIs
- we should try to use standard and appropriate exception types which makes our code cleaner and easier to read and understand
- the most commonly reused exceptions are:
  - `IllegalArgumentException`: non-null parameter value is inappropriate
    - like passing a negative number to an argument expecting number of elements
  - `IllegalStateException`: Object state is inappropriate for method invocation
    - like Object is not initialized
  - `NullPointerException`: parameter value is null where prohibited
  - `IndexOutOfBoundException`: index parameter value is out of range
  - `ConcurrentModificationException`: concurrent modification of object has been detected where it is prohibited
  - `UnsupportedOperationException`: object does not support method

## Item 73
- Throw exceptions appropriate to the abstraction
- when exceptions propagate outward, we may get one that has no apparent connection to the task that it performs.
- to avoid this problem we can do *exception translation* which is basically catching the exception (when appropriate) and throwing a more relevant exception to the higher level instead
- this is an example from `AbstractSequentialList`:
```Java
public E get(int index) {
    ListIterator<E> i = listIterator(index);
    try {
        return i.next();
    } catch (NoSuchElementException e) {
        throw new IndexOutOfBoundsException("Index: " + index);
    }
}
```
- a special form of exception translation is called *exception chaining* which is when lower level exception might be useful to someone debugging the higher level exception. in this case, we pass the lower level exception to the higher level exception:
```Java
try {
    ...
} catch (LowerLevelException cause) {
    throw new HigherLevelException(cause);
}
```
  - most standard exceptions have chaining-aware constructors which pass the "cause" to a higher level constructor which can be accessed later.
  - for exceptions, we can use `Throwable`'s `initCause` method which later can be accessed using `getCause`

**Notes**
- while exception translation is superior to mindless propagation of exceptions from lower layers, it should not be overused

## Item 74
- Document all exceptions thrown by each method
- always declare checked exceptions individually and document precisely the conditions under which each one is thrown
- use JavaDoc's `@throws` tag
- it is also a good practice to document unchecked exceptions that the method can throw. These unchecked exceptions are usually programmer's error and doing this, they will know what to expect and how to properly use that method or interface.

## Item 75
- Include failure-capture information in detail messages
- to capture a failure, the detail message of an exception should contain all the values for all the parameters and fields that contributed to the exception
  - e.g. if we get `IndexOutOfBoundException`, it would be very helpful to have the index value as well as the lower bound and upper bound
  - an exception to this (for security purposes) is passwords, encryption keys and the like which we don't wanna include in the detail message

## Item 76
- Strive for failure atomicity
- it means a failed method invocation should leave the object in the state that it was in prior to the invocation
- there are different ways to achieve this goal. For example in stack implementation from before:
```Java
public Object pop() {
    if (size == 0) {
        throw new EmptyStackException();
    }
    Object result = elements[--size];
    ...
}
```
so here we check the size first and throw exception instead of making size `-1` and then wait for the exception to be thrown when accessing `elements[-1]`
- another way to accomplish failure atomicity is to change the order of the computations if possible, so the part that may cause exception comes first.
- Another approach is to perform operations on a copy of the object and then if everything was successful apply them back to the original
- it should be mentioned that failure atomicity is not always achievable (like to threads without proper synchronization modify something) or sometimes making something atomic will hugely complicate the code. In these cases we should mention it in the API documentation that what is the broken state that may happen by a method invocation for example

## Item 77
- Don't ignore exceptions
- an empty catch block defeats the purpose of exceptions. We should always take proper actions to address exceptions
- in some rare cases it might be ok to ignore them, in which case we should put a comment in catch block with explanations:
```Java
int numColors = 4; // Default
try {
    numColors = getNumColors();
} catch (ExecutionException | TimedoutException ignored) {
    // in this case just use default value
}
```
