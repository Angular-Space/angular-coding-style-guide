# A**ngular Space - Style Guide**

## **Rule:** `.subscribe()` calls

### **What is an Observable?**

Observables are streams of data that emit values over time, they are similar to Promises but emit multiple values rather than just the one. Observable streams can come from a variety of sources, for example an HTTP call or an event like a mouse or keyboard.

To be able to receive data from a stream, we need to `subscribe` to the Observable first, once we have subscribed we will get the current and all subsequent values, the stream of data will continue to emit values until the observable `completes` or an `error` is generated.

The Observable has a callback function to the `subscribe` method, this is called the `observer` and has three optional arguments.

- `next`: Called for each value emitted by the `observable`.
- `error`: Called if the Observable encounters an error.
- `complete`: Called when then Observable finishes emitting values.

---

### **Why we recommend this**

Using `async` to handle the `subscribing` and `unsubscribing` is the recommended approach when using Observables. The `async` pipe handles when the values in the stream have changed and marks the component as `markedForChange` for the change detection to update on the next cycle.

There are times when manually subscribing to an Observable is unavoidable, we may need to manipulate the data being returned before displaying it in the UI, for example.

When manually subscribing to an Observable we need to remember to `unsubscribe` and there are several ways we can do this.

### When to use `.subscribe()`

As mentioned previously, the preferred option is to automatically `unsubscribe` using the `async` method, but when we need to manually `unsubscribe` we have the follow options available and as of Angular 19 these are:

**Async:**

This operator subscribes to an `Observable` returning the latest emitted value. When the component is created, the `async` pipe subscribes to the `observable$` variable and displays the data in the Document Object Model (DOM). When the value changes the component is marked for changes by the `async` pipe and updated in the next change detection cycle.

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

protected observable$ = this.someService.getAll();
```

```html
@for(employee of observable$ | async ){
<div>Employee name: {{ employee.name }}</div>
}
```

**TakeUntilDestroyed:**

This is a recent addition to the Angular Framework and is part of the `@angular/core/rxjs-interop` library and handles the `unsubscribing` automatically. The `takeUntilDestroyed` takes an optional parameter depending on where it's called from. In the example below, the method is called within an injection context and doesn't require the parameter to be passed.

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

constructor {
	this.someService.getAll().pipe(takeUntilDestroyed()).subscribe((data)=>{
        this.employees = data;
    })
}
```

If used outside of an injection context then we need to pass in a `destroyRef` provider.

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

private destroyRef = inject(destroyRef);

ngOnInit(): void {
	this.someService.getAll().pipe(takeUntilDestroyed(this.destroyRef).subscribe((data)=>{
        this.employees = data;
    })
}
```

**Subscription:**

Creates a new `Subscription` we assign our call to the `observable` to get a reference to it so we can call `unsubscribe` later in the `ngOndestroy` life cycle hook.

```typescript
private employees: Employee[];
private subscription : Subscription;

ngOnit(): void {
    this.subscription = this.someService.getEmployees().subscribe((data)=> {
        this.employees = data;
    })
}

ngOnDestroy(): void {
    this.subscription.unsubscribe();
}
```

Or as an array of `subscriptions`

```typescript
private employees: Employee[];
private readonly subscriptions = new Subscription();

ngOnit(): void {
      const subscription = this.someService.getEmployees().subscribe((data)=> {
        this.employees = data;
    });
	this.subscriptions.add(subscription);
}

ngOnDestroy(): void {
    this.subscriptions.forEach(subscription => subscription.unsubscribe());
}
```

**TakeUntil and Subject:**

The `takeUntil` method uses a Subject to handle the `unsubscribing`. In this example we are using the `pipe` to chain other operators, in the example we use the RxJs `takeUntil` method, if there are other operators in the chain, for example `takeUntilDistinct` then ensure `takeUntil` is the last operator in the chain.

```typescript
private employees: Employee[];
private readonly destroy$ = new Subject<void>();

ngOnInit(): void {
	this.someService.getAll().pipe(takeUntil(this.destroy$)).subscribe((data)=>{
        this.employees = data;
    })
}

ngOnDestroy():{
    this.destroy$.next();
    this.destroy$.complete();
}
```

**Take:**

Similar to the `takeUntil` the `take(1)` will unsubscribe, but only when the first value is emitted, so if the component is destroyed before the `take(1)` has happened then the subscription will remain active. Once the value has been emitted it will call `complete` on the Observable. To ensure that the `unsubscribe` will always happen add a subscription and call `unsubscribe` from the `ngOnDestroy` life cycle hook.

```typescript
private employees: Employee[];

ngOnInit(): void {
	this.someService.getAll().pipe(take(1)).subscribe((data)=>{
        this.employees = data;
    })
}

```

**First:**

This operator is like a combined `take(1)` and a `takeWhile`, if `first` is called without a predicate function then it will emit the first value and completes.

```typescript
ngOnInit(): void {
	this.someService.getAll().pipe(first()).subscribe((data)=>{
        this.employees = data;
    })
}
```

If it's called with a predicate function it will emit the first value that passes the condition of the function and completes.

```typescript
ngOnInit(): void {
	this.someService.getAll().pipe(first(data => data.isActive)).subscribe((data)=>{
        this.employees = data;
    })
}
```

Similar to the `take(1)` operator, if the component is destroyed before a value has been emitted then the subscription will remain open. To avoid memory leaks with this method add a `Subscription` and an `ngOnDestroy` life cycle hook.

**TakeWhile:**

This operator requires a predicate function, when the functions criteria has been met then the Observable will complete and unsubscribe.

```typescript
private employees: Employee[];

ngOnInit(): void {
	this.someService.getAll().pipe(takeWhile(data => !data.isActive))
        .subscribe((data)=>{
        this.employees = data;
    })
}
```

If the component is destroyed before a value has been emitted then the subscription will remain open. To avoid memory leaks with this method add a `Subscription` and an `ngOnDestroy` life cycle hook.

**Thought process behind this recommendation:**

---
