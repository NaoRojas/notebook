# Change Detection

🚀 As a frontend framework, Angular strives for the best possible performance.

This means that it needs to re-render the view only when it’s needed — when the bindings change. This is exactly why the concept of Change Detection was created.

The knowledge of how the Change Detection mechanism works will let you write more efficient and performant Angular code. So jump in 😉

# How does Angular Detect the changes?

Each time when the component instance is created — Angular framework creates an associated ***View\*** object. In simple words, in the change detection mechanism “***View\***” stands for “***Component Instance\***”.

Every ***View\*** contains the list of its bindings. Let’s say we have the following Angular component:

![Component Template](https://miro.medium.com/v2/resize:fit:700/1*MG_tzFQeMox-6mR42kqBVA.png)



![img](https://miro.medium.com/v2/resize:fit:700/1*Up0H0fujtmZBewubMdV0Ow.png)



![img](https://miro.medium.com/v2/resize:fit:700/1*BPgYxoVHmiYzakWTDalLgg.png)

This will enforce Angular to create a ***View\*** with the following bindings: *greeting, author.firstName, author.lastName, timestamp.*

*Now, when any of these bindings are changed — Angular will run the change detection cycle to reflect the changes on the view.*

For example, when we click the **“Update”** button — we can see that the timestamp value changes on the view. However, notice that until the button is clicked — the change detection mechanism is NOT triggered at all.

But how does Angular know that the value is indeed changed? This is done by just the JavaScript “**equals**” operator `===` with minor adjustments. If any of the bingings’ value does not match its previous value (change detected)— the view is re-rendered and all the bindings are re-calculated.

# The role of NgZone

Despite the common misconception, **Angular’s change detection mechanism does not depend on NgZone and can nicely work without it being used at all.**

We will take a closer look at its real role. Let’s modify our component a bit:

![img](https://miro.medium.com/v2/resize:fit:700/1*MV5tAuw1ALryUuxViYr49Q.png)

What’s changed here:

1. We change the author’s name in a second delay after the component was initialized.
2. We create a counter that checks how many times the component’s ***View\*** was checked and log the value to the console.

So what is the result?

![img](https://miro.medium.com/v2/resize:fit:700/1*ictot4Q8rRN0BqrsXEad9g.png)

We see that the name is indeed changed but there are 3 view checks in place. So what is happening here?

1. **The first check** is done when the component is ready to be rendered (right after the ngAfterViewInit hook). It is needed for Angular to know the initial values of component’s bindings.
2. **The second check** is done afterwards to make sure that the values have not been changed after the previous check.
3. **The third check** appears with the delay and is triggered by the change in ‘*author.firstName*’ value. Here ngZone takes the stage! It creates a dedicated “zone” for the Angular app and notifies about any asynchronous changes. It can be *setTineout/setInterval* call like in our case or any other operations that are done in the **asynchronous** manner: *user input, click, eventListener or XMLHTTP request.*

Note that NgZone triggers the change detection even in the cases when no changes should be reflected on the UI. For example, if we make an HTTP call to load something asynchronously but the data response data is bound to the View — *Angular will anyway run the additional change detection checks.*

Let’s say we update our timed-out logic not to change the UI-bound “*author.firstName*” but simply log the hardcoded string to the console.

![img](https://miro.medium.com/v2/resize:fit:700/1*XKCr9gypPF_grv94mKNL0A.png)

We will still see that 3 checks were fired despite **nothing on UI can be changed!**

![img](https://miro.medium.com/v2/resize:fit:700/1*njfvCHJoeHS3VCR6PqxU4g.png)

# How to avoid false-positive NgZone triggers

The rule of a thumb is to run all of your non-UI-bound asynchronous logic outside of NgZone. Angular provides the NgZone injectable for this purpose. Let’s use it and see the result:

![img](https://miro.medium.com/v2/resize:fit:700/1*N-kmi1klbKcJNsp3PXS3Dg.png)

Extracting the asynchronous context outside of NgZone

Now we see that there extra trigger disappears:

![img](https://miro.medium.com/v2/resize:fit:700/1*Ncmf5NGoRiciTi6Ut699Ww.png)

# Make Your Angular App 100+ Times Faster with ‘OnPush’ Change Detection Strategy

## How does Angular mark the View for check?

By default, Angular uses its Default Change Detection strategy `ChangeDetectionStrategy.Default`.

It waits for the triggering events (clicks, keyboard inputs, bindings changes, timers etc) and **checks for changes ALL THE COMPONENTS in the component TREE from top to bottom**.

![img](https://miro.medium.com/v2/resize:fit:600/1*z3NR45ds3DBBytzlNn_DGA.gif)

It is also known as ***dirty checking\*** as far as each node in the tree goes through the checks even if they are not needed.

While dirty checking is really high-performant and handles thousands of checks per second, it can lead to performance issues when:

- the component tree becomes too complex;
- triggering events occur too often;

Luckily, there is a way to reduce both of the above problems by using another Change Detection Strategy — `ChangeDetectionStrategy.OnPush`.

# How does ChangeDetectionStrategy.OnPush work?

Enabling `OnPush` Change Detection Strategy is just a one-line change:

![img](https://miro.medium.com/v2/resize:fit:700/1*VRgM-JFInm6r-I2jRGkr-g.png)

Enabling ChangeDetectionStrategy.OnPush

This mode forces the framework to **check the component and its children only in the following cases:**

1. **@Input’s** *reference* is changed (comparing by “===” operator)
2. The event is triggered by the component or one of its children (by **@Outputs** or **@HostListeners**)
3. The *“**async**”* pipe gets a new value from the Observable
4. Change detection is triggered manually

![img](https://miro.medium.com/v2/resize:fit:600/1*6cFRoSYEhaJ3xpKLXxkBmw.gif)

**OnPush** prevents Angular from unnecessary checks

From the diagram above you can see that by using `OnPush` in just a single component we significantly reduced amount of work that framework does on every single check.

What actions **do NOT trigger** change detection mechanism:

1. Timers: `setInterval`, `setTimeout` ;
2. Values emitted by **Promises** and **Observables**;
3. DOM events that are not listened to by Angular;
4. @**Inputs** value change **with the same reference**;

# Practice

Let’s take a look at the component created in [the previous article about change detection](https://medium.com/@yar.dobroskok/angular-change-detection-explained-shortly-ngzone-basics-be0719fa8d99) and modify it to showcase how powerful `OnPush` strategy is.

## Case 1

![img](https://miro.medium.com/v2/resize:fit:700/1*r10MOs4w6FyFTU9huKtFhA.png)

Component template

![img](https://miro.medium.com/v2/resize:fit:700/1*Rr5VhGujxtwAdbaqeBI2SA.png)

Component code

**Code overview:**

1. When the component class is instanciated — we already have our fields `greeting`, `author` and `timestamp` set with the initial values.
2. After that in `ngOnInit` we have the 3 asynchronous actions that set new values for our component fields.
3. Note that 4th one calls the `detectChanges` method of Angular’s change detector in 5 seconds.

**So what is the result?**

![img](https://miro.medium.com/v2/resize:fit:700/1*uOH3ms0n2VyAvp4XFTlc5w.gif)

We can see that all the 3 async actions are triggered and each of them changes one of the component’s bindings. However, **we do not see any changes rendered on the UI** until the `detectChanges()` is called.

And then all the changes that we had made before are rendered simultaneously *in one single check.*

You can now analyze why all of the above happens. If you need a hint — scroll up to see what actions do NOT trigger change detection with `OnPush`.

**Benefit:** We can use `OnPush` to skip unnecessary checks. Especially for cases:

1. When we want to skip rendering of particular values.
2. When we load the data from different sources and want to show only the end result.

## Case 2

Let’s look at the other example showing the crucial performance improvement when using a component with `@Input`s.

**Parent component:**

![img](https://miro.medium.com/v2/resize:fit:700/1*Jz_Ah1UVDqfhu3u0ugx1zA.png)

Parent component code

![img](https://miro.medium.com/v2/resize:fit:700/1*Z761GGyOlXI5XlNtPSazfg.png)

Parent component template

Parent component contains the person’s data. Let’s assume we get this data from an HTTP call.

On the template we only display the child component and 2 buttons. Both of them change the person’s name. However, one of them updates fields values and the other one replaces object’s reference.

**Child component:**

![img](https://miro.medium.com/v2/resize:fit:700/1*oJ7YoyGKY6SRPp6QNzPqHA.png)

Child component code

Child component is a typical “presentational” component that does nothing but accepts input from the parent. Note that it uses `OnPush` change detection strategy.

Let’s see the result:

![img](https://miro.medium.com/v2/resize:fit:700/1*HRRbLYszIksh1cpIPlZXxQ.gif)

Here we observe that in case when the `Input`s properties change — **Angular does not detect these changes**.

This is happening because by enabling the `OnPush` strategy we tell the framework: *“Hey, do not check this input until the object reference is changed”*.

**This gives us significant performance** **improvement** because in the real-world apps we deal with the objects that with time get pretty heavy. And the only way for Angular to detect that something is changed within the object **is to check each of its properties in each of the check cycles.** You can only imagine the amount of work it does when its time to check dozens of components on the view and each of them usually has many imports inside.

Let’s say we received an Input with **30** properties and each of them has in average **3** fields that are objects with just **10** properties inside. How much comparisons Angular needs to do on each check?

> \# of Comparisons = 30 x 3 x 10 = 900 !

And I bet it’s not even the worst scenario you’ve ever seen in your project 😉

And how many comparisons does Angular need to make to know that changes happened with `OnPush` option?

**It’s only one check!** `oldValue === newValue`. By enabling `OnPush` strategy we decreased the complexity of each check by 900 times.

And additional bonus — the quantity of checks is also decreased.

**Benefit:** By using `OnPush` strategy when the component accepts the non-primitive inputs from parent — we get significant performance increase because of multiplied effect of:

1. Decreasing each single check complexity;
2. Decreasing the total number of checks;

# Summary

As you have seen — `OnPush` change detection strategy is a very powerful tool to dramatically increase Angular application performance.

In my work I use it as a default option in 95% of cases. **Especially it perfectly fits** the components that:

- are “presentational” / “dumb” components;
- contain complex asynchronous data flow;
- accept massive `Input`s;
- have many simultaneous instances rendered on the view;
- you want to have control of re-render flow;