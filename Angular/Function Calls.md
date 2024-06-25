# Why you should never use function calls in Angular template expressions

Angular templates are awesome and extremely powerful.

With structural directives and property bindings, we are equipped to create even the most complex views with very clear semantics:

```html
<ng-container *ngIf="isLoggedIn">
  <h1>Welcome {{ fullName }}!</h1>
</ng-container>
```

Because expression are so powerful, they can easily grow in complexity when our views become more complex.

Before we know it, we end up with function calls in our Angular templates:

```html
<ng-container *ngIf="isLoggedIn">
  <h1>Welcome {{ fullName }}!</h1>
  <a href="files" *ngIf="hasAccessTo('files')">Files</a>
  <a href="photos" *ngIf="hasAccessTo('photos')">Photos</a>
</ng-container>
```

While function calls in Angular templates are super convenient and technically valid, they may cause serious performance issues.

This article explains why and how to work around the performance problems.

# The problem

For the sake of demonstration, let’s assume we have a `PersonComponent` that uses a function call `fullName()` in its template to display the full name of a person that is passed in via the `person` property:

```js
@Component({
  template: `
    <p>Welcome {{ fullName() }}!</p>
    <button (click)="onClick()">Trigger change detection</button>
`
})
export class PersonComponent {  
  @Input() person: { firstName: string, lastName: string };
  
   constructor() { }  
  
  fullName() {
    return this.person.firstName + ' ' + this.person.lastName
  }  
  
  onClick() {
    console.log('Button was clicked';
  }}
```

Here, the `fullName()` function is executed every time Angular change detection runs. **And that can be many times!**

**Every time the button is clicked**, the `fullName()` function is executed.

While clicking a button may sound harmless, suppose that six months later the requirements for our PersonComponent change and we now need to deal with more events that trigger change detection:

```js
@Component({
  template: `
    <p>Welcome {{ fullName() }}!</p>
    <div (mousemove)="onMouseMove()">Drop a picture here</div>
`
})
export class PersonComponent {
  @Input() person: { firstName: string, lastName: string };
  
  constructor() { }  
  fullName() {
    return this.person.firstName + ' ' + this.person.lastName
  }  
  onMouseMove() {
    console.log('Mouse was moved');
  }}
```

Suddenly, the `fullName()` function is executed hundreds of times when the mouse is moved over the `div`.

And because the `fullName()` code was already written six months before, we may not realise the impact of our new code.

Furthermore, change detection can be triggered outside of the `PersonComponent`:

```html
<person [person]="person"></person>
<button (click)="onClick()">
  Trigger change detection outside of PersonComponent
</button>
```

Here, the `fullName()` function inside `PersonComponent` is executed every time the button outside of `PersonComponent` is clicked.

So why does this happen? And what can we do about it?

# Why are functions in expressions called so many times?

The goal of Angular change detection is to figure out which parts of the UI need to be re-rendered when changes occur.

To determine whether `<p>Welcome {{ fullName() }}!</p>` needs to be re-rendered, Angular needs to execute the `fullName()` expression to check if its return value has changed.

Because Angular cannot predict whether the return value of `fullName()` has changed, it needs to execute the function every time change detection runs.

So if change detection runs 300 times, the function is called 300 times, even if its return value never changes.

Depending on the logic inside the function, running a function hundreds of times can become a serious performance issue.

This becomes painfully hidden in templates when getters are used:

```js
@Component({
  template: `
    <p>Welcome {{ fullName }}!</p>
`
})
export class PersonComponent {  
  @Input() person: { firstName: string, lastName: string }; 
  constructor() { }  
  get fullName() {
    return this.person.firstName + ' ' + this.person.lastName
  }}
```

Here, the template shows no visual indication of any function call being made but the `get fullName()` getter is called every time change detection runs.

# What about ChangeDetectionStrategy.OnPush?

When we enable `ChangeDetectionStrategy.OnPush` for the `PersonComponent`, we tell Angular change detection to ignore changes outside `PersonComponent` that do not affect its input properties.

As a result, change detection inside `PersonComponent` only runs when one of its input properties change, in this case its `person` property.

In the following example:

```
<person [person]="person"></person>
<button (click)="onClick()">
  Trigger change detection outside of PersonComponent
</button>
```

the `fullName()` function inside `PersonComponent` is no longer executed when the button outside of `PersonComponent` is clicked.

Brilliant! Doesn’t that solve all our potential performance issues?

No, unfortunately not.

**Why not?**

Because the `fullName()` function is still executed every time change detection is initiated inside the `PersonComponent` itself:

```js
@Component({
  template: `
    <p>Welcome {{ fullName() }}!</p>
    <div (mousemove)="onMouseMove()">Drop a picture here</div>
`
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class PersonComponent {  
  @Input() person: { firstName: string, lastName: string }; 
  constructor() { }  fullName() {
    return this.person.firstName + ' ' + this.person.lastName
  }
  onMouseMove() {
    console.log('Mouse was moved');
  }}
```

Here, the `fullName()` function is still executed every time the mouse moves over the `div` , even if we have `ChangeDetectionStrategy.OnPush` enabled.

So while `ChangeDetectionStrategy.OnPush` can help us ignore change detection cycles from outside, the functions in our expressions are still being executed every time change detection is triggered from within the component itself.

So how can we prevent unnecessary function calls?

# The solution: how to avoid unnecessary function calls

## Strategy 1— Pure pipes

A first strategy to reduce function calls is to use pure pipes.

By telling Angular that a pipe is pure, Angular knows that the pipe’s return value does not change if the pipe’s input does not change.

For our existing example, we can create a pure pipe to calculate a person’s full name:

```js
import { Pipe, PipeTransform } from '@angular/core';@Pipe({
  name: 'fullName',
  pure: true
})
export class FullNamePipe implements PipeTransform {
  transform(person: any, args?: any): any {
    return person.firstName + ' ' + person.lastName;
  }
}
```

A pipe in Angular is pure by default, so we can skip the `pure: true` part in the pipe decorator:

```js
import { Pipe, PipeTransform } from '@angular/core';@Pipe({
  name: 'fullName'
})
export class FullNamePipe implements PipeTransform {
  transform(person: any, args?: any): any {
    return person.firstName + ' ' + person.lastName;
  }
}
```

We can consume the pipe by adding it as a declaration to our module:

```js
@NgModule({
  declarations: [ AppComponent, PersonComponent, FullNamePipe ]
})
export class AppModule { }
```

and using it in the view of our `PersonComponent`:

```js
@Component({
  template: `
    <p>Welcome {{ person | fullName }}!</p>
    <div (mousemove)="onMouseMove()">Drop a picture here</div>
`
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class PersonComponent implements OnChanges {@Input() person: { firstName: string, lastName: string };constructor() { }onMouseMove() {
    console.log('Mouse was moved');
  }}
```

Angular is now smart enough to know that the expression `{{ person | fullName }}` does not change if the `person` does not change.

As a result, Angular skips the execution of the pipe’s `transform` method if the person does not change.

But what if our logic is more complex and cannot be handled by a pipe?

## Strategy 2— Manually calculate the values

A second strategy to avoid unnecessary function calls is to manually calculate the values we need in our view from within the component’s controller.

As a developer, we can take advantage of the fact that we know when a value of an expression can change.

In our example, we know that the full name can only change when the `person` changes, so we can add a `fullName` property to the component and recalculate its value in `ngOnChanges` only when the value of the component’s `person` input changes.

That leaves us with a simple property that contains the value we need so we can replace the `{{ fullName() }}` function call expression in the view with a `{{ fullName }}` property expression:

```js
@Component({
  template: `
    <p>Welcome {{ fullName }}!</p>
    <div (mousemove)="onMouseMove()">Drop a picture here</div>
`
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class PersonComponent implements OnChanges {
  
@Input() person: { firstName: string, lastName: string };
fullName = '';
constructor() { }
  
ngOnChanges(changes: SimpleChanges) {
    if (changes.person) {
      this.fullName = this.calculateFullName();
    }
  }
                                                   calculateFullName() {  
    return this.person.firstName + ' ' + this.person.lastName;
  }
onMouseMove() {
    console.log('Mouse was moved');
  }}
```

As a result, the `fullName` property is recalculated only when the component’s `person` input changes.

When we hover the `div`, the change detection no longer executes the logic to calculate the full name.

See how this simple change reduces 294 function calls to only 5 function calls when we navigate through 5 person objects and trigger a mousemove event:

![img](https://miro.medium.com/v2/resize:fit:700/1*CGI61ouySrf-YX52KztArg.gif)

Visually, there is no difference. But behind the scenes, the impact of our change is enormous.

You can play with the demo yourself right here: https://stackblitz.com/edit/angular-hxmb6q

# Summary

In this article, we learned that function calls in expression can cause serious performance issues that may go unnoticed and cause your application to become slow, even with `ChangeDetectionStrategy.OnPush` enabled.

To avoid issues, **it is highly recommended not to use function calls in Angular template expressions**.

Instead, you can:

- **use pure pipes** to let Angular know that execution of a pipe can be skipped safely if the pipe’s input does not change
- **manually calculate** the values you need in your component’s controller and recalculate them only when needed

To witness the impact on performance, you can play with a live demo right here: https://stackblitz.com/edit/angular-hxmb6q

Next time you find yourself writing a function call in a template, make sure to think about the potential consequences.

And if you spot a fellow developer using a function call in an Angular template expression during a code review, kindly send them a link to this article to share your knowledge with them.

Because [performance matters](https://developers.google.com/web/fundamentals/performance/why-performance-matters/)!



# Links



https://medium.com/angular-in-depth/how-to-improve-angular-performance-by-just-adding-just-8-characters-877bde708ddd