# **Angular Space - Style Guide**

## **Rule:** `.subscribe()` calls

**What `subscribe()` does.**

Subscribe is the method used to receive values from Observables. An Observable is a continuous stream of data (or events) and by subscribing we are listening to that stream and until we subscribe no data will be received.

**Executes the Observable:** The Subscribe triggers the execution of the Observable and starts generating values, this could be an HTTP request or listening to events.

**Registers Observers:** There are three observer functions to the `subscribe()` method and defines how to handle the events emitted by the Observable.

- `next`: This is called for each new value emitted by the `observable` and where we can process data from the stream.
- `error`: This is called if the Observable encounters an error and provides a way to handle errors.
- `complete`: This is called when the Observable completes, no further values will be emitted and the stream ends.

**Returns a subscription:** The `subscribe()` method returns a Subscription object and is the connection to the Observable, this connection provides a way to `unsubscribe()` from the Observable.

**How to use `subscribe()`:**

```typescript
import { Observable, of } from 'rxjs';

const obs = of(1, 2, 3);

const subscription = obs.subscribe({
  next: (value) => console.log('Received values', value),
  error: (error) => console.error('An error occured', error),
  complete: () => console.log('Complete'),
});

subscription.unsubscribe();
```

**Explanation:**

The `obs` variable creates an Observable which will emit values 1,2 and 3 and then complete.

The `obs.subscribe(...)` starts the subscription.

The `next` handler receives each of the emitted values.

The `error` handler handles any errors that may occur.

The `complete` handler completes the subscription.

The `subscription.unsubscribe()` unsubscribes from the Observable. This part is important, forgetting to unsubscribe will create memory leaks in your Angular applications.

When unsubscribing from an Observable it is often done in the `ngOnDestroy()` lifecycle hook.

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { SomeService } from './someservice';
import { Subscription } from 'rxjs';

@Component({...})
export class SomeComponent implements onInit, onDestroy {
	private subscription: Subscription;
	private readonly someService = inject(SomeService);

	ngOnInit(): void {
    	this.subscription = this.someService.getSomeData().subscribe({
        	next: (value) => console.log('Received values',value),
   			error: (error) => console.error('An error occured', error),
   			complete: () => console.log('Complete'),
        }
  	}

	ngOnDestroy(): void {
    	if(this.subscription) {
        	this.subscription.unsubscribe();
    	}
  	}
}
```

This is just one example of creating a subscription and unsubscribing from it, the following are examples of other variations used to subscribe/unsubscribe from an Observable.

<div style="display: flex; flex-direction: row; justify-content: center; align-items: center; height:40px; background: lightgreen; border-radius: 20px; color:#000; width: 150px;">Recomended</div>

**Async:** Use the `async` pipe to get an Observable stream of data. The `async` pipe handles when the values in the stream have changed and marks the component as `markedForChange` for the change detection to update on the next cycle.

```typescript
import { Component, Async } from '@angular/core';
import { EmployeeService } from './someservice';

@Component({...})
export class EmployeeComponent {
	protected employee$ = inject(EmployeeService).getSomeData();
}
```

```html
@for(employee of employee$ | async; track employee.id ){
<div>Employee name: {{ employee.name }}</div>
}
```

<div style="display: flex; flex-direction: row; justify-content: center; align-items: center; height:40px; background: lightgreen; border-radius: 20px; color:#000; width: 150px;">Recomended</div>

**TakeUntilDestroyed:** When we need to manually subscribe to an Observable we can use the new `takeUntilDestroyed` method this is a recent addition to the Angular Framework and is part of the `@angular/core/rxjs-interop` library and handles the `unsubscribing` automatically. The `takeUntilDestroyed` takes an optional parameter depending on where it's called from. In the first example below, the method is called within an injection context and doesn't require the parameter to be passed.

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

constructor {
	this.someService.getAll().pipe(takeUntilDestroyed()).subscribe({
        next: (value) => console.log('Received values',value),
   		error: (error) => console.error('An error occured', error),
   		complete: () => console.log('Complete'),
    })
}
```

If used outside of an injection context then we need to pass in a `destroyRef` provider.

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

private destroyRef = inject(destroyRef);

ngOnInit(): void {
	this.someService.getAll().pipe(takeUntilDestroyed(this.destroyRef).subscribe({
        next: (value) => console.log('Received values',value),
   		error: (error) => console.error('An error occured', error),
   		complete: () => console.log('Complete'),
    })
}
```

**TakeUntil and Subject:** The `takeUntil` method uses a Subject to handle the `unsubscribing`. In this example we are using the `pipe` to chain other operators, in the example we use the RxJs `takeUntil` method, if there are other operators in the chain, for example `takeUntilDistinct` then ensure `takeUntil` is the last operator in the chain.

```typescript
private employees: Employee[];
private readonly destroy$ = new Subject<void>();

ngOnInit(): void {
	this.someService.getAll().pipe(takeUntil(this.destroy$)).subscribe({
        next: (value) => console.log('Received values',value),
   		error: (error) => console.error('An error occured', error),
   		complete: () => console.log('Complete'),
    })
}

ngOnDestroy():{
    this.destroy$.next();
    this.destroy$.complete();
}
```

**Subscription:** Creates a new `Subscription` we assign our call to the `observable` to get a reference to it so we can call `unsubscribe` later in the `ngOndestroy` life cycle hook.

```typescript
private employees: Employee[];
private subscription : Subscription;

ngOnit(): void {
    this.subscription = this.someService.getEmployees().subscribe({
    	next: (value) => console.log('Received values',value),
   		error: (error) => console.error('An error occured', error),
   		complete: () => console.log('Complete'),
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
      const subscription = this.someService.getEmployees().subscribe({
        next: (value) => console.log('Received values',value),
   		error: (error) => console.error('An error occured', error),
   		complete: () => console.log('Complete'),
    });
	this.subscriptions.add(subscription);
}

ngOnDestroy(): void {
    this.subscriptions.forEach(subscription => subscription.unsubscribe());
}
```

**Take:** Similar to the `takeUntil` the `take(1)` will unsubscribe, but only when the first value is emitted, so if the component is destroyed before the `take(1)` has happened then the subscription will remain active. Once the value has been emitted it will call `complete` on the Observable. To ensure that the `unsubscribe` will always happen add a subscription and call `unsubscribe` from the `ngOnDestroy` life cycle hook.

```typescript
private employees: Employee[];

ngOnInit(): void {
	this.someService.getAll().pipe(take(1)).subscribe({
        next: (value) => console.log('Received values',value),
   		error: (error) => console.error('An error occured', error),
   		complete: () => console.log('Complete'),
    })
}
```

**First:** This operator is like a combined `take(1)` and a `takeWhile`, if `first` is called without a predicate function then it will emit the first value and completes.

```typescript
ngOnInit(): void {
	this.someService.getAll().pipe(first()).subscribe({
        next: (value) => console.log('Received values',value),
   		error: (error) => console.error('An error occured', error),
   		complete: () => console.log('Complete'),
    })
}
```

If it's called with a predicate function it will emit the first value that passes the condition of the function and completes.

```typescript
ngOnInit(): void {
	this.someService.getAll().pipe(first(data => data.isActive)).subscribe({
        next: (value) => console.log('Received values',value),
   		error: (error) => console.error('An error occured', error),
   		complete: () => console.log('Complete'),
    })
}
```

Similar to the `take(1)` operator, if the component is destroyed before a value has been emitted then the subscription will remain open. To avoid memory leaks with this method add a `Subscription` and an `ngOnDestroy` life cycle hook.

**TakeWhile:** This operator requires a predicate function, when the functions criteria has been met then the Observable will complete and unsubscribe.

```typescript
private employees: Employee[];

ngOnInit(): void {
	this.someService.getAll().pipe(takeWhile(data => !data.isActive))
        .subscribe({
        next: (value) => console.log('Received values',value),
   		error: (error) => console.error('An error occured', error),
   		complete: () => console.log('Complete'),
    })
}
```

If the component is destroyed before a value has been emitted then the subscription will remain open. To avoid memory leaks with this method add a `Subscription` and an `ngOnDestroy` life cycle hook.
