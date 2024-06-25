# DI Resolution modifiers

**DI Resolution modifiers** is the decorator we use for Dependency Injection.

```
constructor(@Inject(Document) private document: HTMLDocument) {}orconstructor(@Optional() private optionalService: OptionalService) {}
```

You might have used it before, or just seen on some library and never used it before. Either ways, you can learn what that is and how angular manages dependencies by one by one in this article.

## | What @Inject means?

```
constructor(private messageService: MessageService) {}
```

Most time we declare dependencies of the class like that and angular detect which service or instance it is depends on and takes care of creating and injecting the one. So when angular sees that clause, it knows the class should receive the MessageService and tries to find the one.
But sometimes this declaration does not work, like the case below, you want to inject specific value with some meaning, it is hard for angular to figure out what dependency it should inject.

```
constructor(private isDev: boolean) { 
  // what should it refer to???
}
```

**@Inject** decorator is the manual indicator tells angular what it should look for. Dependency is composed with **provide** and **valueFactory(useValue, useExisting, useFactory etc..)** parts and InjectionToken class is used to as the provide to make it distinguishable.

![img](https://miro.medium.com/v2/resize:fit:601/1*nwN8cN7-2QABg27dalwBjQ.png)

Inject takes any type, which means this is also possible. (But deprecated)

## | How Injector manages dependencies.

Dependency Injection uses Injector mechanism so it is important for us to know the concept of Injector.

[[publish\] injector - StackBlitzA angular-cli project based on @angular/animations, @angular/compiler, @angular/core, @angular/common…stackblitz.com](https://stackblitz.com/edit/angular-ivy-g1gswo?embed=1&file=src/app/app.component.ts)

Angular provides 2 types of injector, one is **module Injector** and the other is **Element Injector**. ([link](https://angular.io/guide/hierarchical-dependency-injection#two-injector-hierarchies))

```
@Injectable()
export class AppService {}@Component({
  ..., 
  providers: [AppService]
})
export class AppComponent {}@NgModule({providers: [AppService]})
export class AppModule {}
```

Even though AppService is one, depends on whether it is provided in module level or component level, they receive different type of parent injector.

![img](https://miro.medium.com/v2/resize:fit:700/1*l4eiuLqp3PFX0QvD3OWpVw.png)

same service gets different injector type by where it is created.

[[publish\] injection tree - StackBlitzA angular-cli project based on @angular/animations, @angular/compiler, @angular/core, @angular/common…stackblitz.com](https://stackblitz.com/edit/angular-ivy-x8umxh?file=src/app/app.component.ts)

Element Injector and Module Injector form separate trees of injector. Injector instance has the prop referencing parent and angular traverse Injector tree up until it finds the dependency and gives you the closest one. Otherwise it reaches the first Injector which is nullInjector and return the error.
(It traverse element Tree first, then does module one.).

![img](https://miro.medium.com/v2/resize:fit:700/1*QtCxAvSRvtDai-33bUNF9Q.png)

means it reached to the first injector and couldn’t find the dependency

Now enough of DI, we can back to the modifier and check one by one.

# Modifiers

## - @Optional

When you create libraries or some class for lazy modules, you can’t assume that every service would exist at the time of boosting. Angular throws an error when it fails to retrieve the dependency, and @**Optional modifier** prevents the error from happening. (It just prevent injection error, and the responsibility of using it is on you.)

```
constructor(@Optional() private appService: AppService) {
}// you should check whether the dependency is injected or not manually.
getAppServiceValue(){ 
  return this.appService ? this.appService.value : 0;
}
```

## - @Self

Besides Optional, @Self is the next one to use often. In case you don’t want your service to receive dependency from above but from same level.

Let’s say you made a LayerService that shows some contents and this layer might close differently by it has button or Not (controlled by LayerButtonService).
Layer might have layer inside so we can’t receive any LayerButtonService.
Yes, we can make layerService to receive optionally layerButtonService only it is injected in the same level.

```
@Injectable()
export class LayerService { 
  constructor(@Optional() @Self buttonService: LayerButtonService){}
}@Component({
  selector: 'layer', 
  providers: [LayerService], 
  ...
})
export class LayerComponent { .... }@Component({
  selector: 'dropdown', 
  providers: [LayerButtonService, LayerService], 
  ...
})
export class DropdownComponent { .... }
```

Also at Directive, you can check the directive is attached to specific class and maybe runs different logic.

## - @SkipSelf

SkipSelf is opposite of the Self. it will check dependencies declared on the parents injector.
You can check depth or other values to check accumulation.

```
@Injectable()
export class LayerService {
  public depth: number;
  constructor(@Optional() @SkipSelf() parentLayer: LayerService) {
    this.depth = parentLayer ? parentLayer.depth + 1 : 0;
  }
}@Component({
  selector: 'layer',
  template: '<div>layer depth {{depth}}<layer [nth]="nth - 1" *ngIf="nth > 0"></layer></div>',
  providers: [LayerService],
})
export class LayerComponent {
  @Input() nth = 0;
  depth;
  constructor(public layerService: LayerService) {
    this.depth = layerService.depth;
  }
}
```

[[publish\] skipSelf - StackBlitzA angular-cli project based on @angular/animations, @angular/compiler, @angular/core, @angular/common…stackblitz.com](https://stackblitz.com/edit/angular-ivy-vnsmab?file=src/app/app.component.ts)

## - @Host

Host retrieve dependencies from host element or component. So in some cases it is similar to @Self in the component level, but with other concepts it shows different outcome. I really does not have a chance to use it, but in case you want to get the component injector from directive, you can us