---
title:  "Android Architecture Components: DataBinding - Dependent Properties"
date:   2018-07-27 15:00:00
categories: [android]
tags: [android, databinding, kotlin]
---

Ever heard of DataBinding? I myself would like to live in a world where every Android developer knows about it. A world where the concept of findViewById and handwritten boilerplate glue code does not exist. Where you can easily tie your data to the UI and forget about updating it in case the data changes.

![](https://cdn-images-1.medium.com/max/3188/1*yPEGoPdhhor21NGLHjxXHg.png)

To achieve this, my fellow developers, I decided to spread the word about a very powerful feature of DataBinding: **DEPENDENT PROPERTIES.**

*“What are dependent properties?”* you may ask*.*

About a year ago, I found [this](https://medium.com/google-developers/android-data-binding-dependent-properties-516d3235cd7c) short article by [George Mount](https://medium.com/@georgemount007) which introduced a really neat feature of DataBinding * *drrrrrrrrrrrrrrrrrum-roll ** the **Bindable properties** which depend on other **Bindable properties**.

The cool thing about dependent properties is that any change in dependencies will automatically trigger an update to the field that declared them as dependencies. This is really powerful on screens where you have input views that rely upon the state of other views.

Let’s look at an example, so you can easily get the gist of it and see how useful this can be when implementing a reactive UI.

For the sake of simplicity I chose to have a first name and a last name as input fields, format them somehow and display the result. In the followings I’ll refer to this as the **“displayName”**.

This is how it looks in action:

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/Q4x5z9VzrTE" frameborder="0" allowfullscreen></iframe></center>

*But how can this be implemented? *Fair question. I’ll show how I did it.

If you’ve gotten this far, I assume you’re familiar with the [DataBinding library](https://developer.android.com/topic/libraries/data-binding/), so I won’t explain how to set it up, but if you’d like to dust off your knowledge here’s a [guide](https://developer.android.com/topic/libraries/data-binding/start) on how to do it.

As a first step I created the layout file.

<script src="https://gist.github.com/ArthurNagy/db500249f9d00c8ef8c11e721a40e0bb.js"></script>

What you’ve just seen is a short version of the layout. I’ve left out the irrelevant tags to focus on what’s important right now. I declared a variable, the **view model **that stores and handles the data, then bound the **displayName** property from within the ViewModel to the **TextView**. Next, the **firstName** and **lastName** properties were bound using the [two-way data binding](https://medium.com/google-developers/android-data-binding-lets-flip-this-thing-dc17792d6c24) syntax to the **EditText**s (meaning that any changes made in the **EditText**s, will show up in the **ViewModel**’s String properties too).

Following this, in the view model class, I declared the **displayName** bindable property to depend on the **firstName** and **lastName** bindable properties, by enumerating them in the **@Bindable** annotation as follows:

<script src="https://gist.github.com/ArthurNagy/1a9701898adedb4b5fa7ee5fe0c1feca.js"></script>

To use the **notifyPropertyChanged** method which signals whenever the @Bindable properties change, the ViewModel had to extend the **BaseObservable** class. With this, every time the **firstName** & **lastName** changes** **the **notifyPropertyChanged **is called. This updates the **displayName**, which on its turn updates the UI with the new value.

To wrap this up, as a final step, I tied the **UserViewModel** instance to the XML layout:

<script src="https://gist.github.com/ArthurNagy/fc3080cc57406e3b6529204df76cab60.js"></script>

In the activity class I created a binding instance and set a **UserViewModel** to it.

Aaaaand that’s it.

Or is it? 🤔

Well, not exactly. I mean it’s nice and everything, but as you can see I filled the UserViewModel with boilerplate code in order to make this work.

Luckily, not so long ago I stumbled upon [this](https://youtu.be/0cjr4K0tqs4) interesting talk by [Lisa Wray](undefined) about using DataBinding together with Kotlin. She used Kotlin’s [delegated properties](https://kotlinlang.org/docs/reference/delegated-properties.html) feature to create a custom delegate for bindable fields, making the code more readable this way. Following her steps, this is what I ended up having:

<script src="https://gist.github.com/ArthurNagy/90101fe819c41e79e7fa68d1fd5d3222.js"></script>

The **BindableDelegate **has a receiver of **R** which extends **BaseObservable** and handles any kind a property of type **T. **In the constructor of the delegate it expects an initial value of the bindable property as well as its **binding resource **identifier. So now, when a value is set, **notifyPropertyChanged **will be called on the receiver class.

After rewriting the ViewModel to use the custom delegate I had something like this:

<script src="https://gist.github.com/ArthurNagy/dba7c3fe72386ee7e0a18f414dc4d775.js"></script>

All right, now the code is clean and concise. It can’t get any better, I’m pretty sure!

<iframe src="https://giphy.com/embed/hPPx8yk3Bmqys" width="480" height="435" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/request-donald-wrong-hPPx8yk3Bmqys">via GIPHY</a></p>

What seems to be the problem?

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/BRczNG2uNM4" frameborder="0" allowfullscreen></iframe></center>

Turns out, the data doesn’t survive configuration changes such as screen rotations or entering multi-window mode.

So that’s where [Android Architecture Components: ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel) came into play. ViewModel is designed to store and manage UI-related data in a lifecycle conscious way. *Well then, let’s extend it, use it and problem solved, right?* It’s not exactly that simple, the UserViewModel already extends BaseObservable, in order to use bindable properties.

Since extending multiple classes is not possible, my options were limited. I could have extended the Architecture Components provided ViewModel and copy-paste the whole code from the BaseObservable class. It’s doable, but let’s be honest, that’s an ugly solution, so I chose not to pursue it.

Fortunately, the DataBinding library provides Observable classes, more specifically an [ObservableField](https://developer.android.com/reference/android/databinding/ObservableField)(or it’s primitive variants) which behaves like a BaseObservable but also wraps a value. Instances of this class can be used as properties in the view model instead of the @Bindable properties. This way properties can be bound to the view and retain the data on configuration change events by extending the ViewModel class.

Mmmkay, but what about the dependent properties? How can one deliver this feature with ObservableField? In the [Android Studio 3.1](https://developer.android.com/studio/releases/#codingide) update, Google silently updated the DataBinding library, and one of the changes was:
> The [ObservableField](https://developer.android.com/reference/android/databinding/ObservableField.html) class can now accept other [Observable](https://developer.android.com/reference/android/databinding/Observable.html) objects in its constructor.

Crystal clear, right? It turns out, using this, the same dependent property behaviour can be achieved as with @Bindable properties. So I refactored the UserViewModel to use the Architecture Component ViewModel and ObservableFields as properties:

<script src="https://gist.github.com/ArthurNagy/e5aa9c92f364f70fa30063558c7c0522.js"></script>

To make the code more readable, I created a top-level inline function which hides the ObservableField instantiation and overrides the get() method:

<script src="https://gist.github.com/ArthurNagy/7a203550024fe77851163659d6fd6061.js"></script>

I also modified the view model creation logic in the activity:

<script src="https://gist.github.com/ArthurNagy/1844133b8f376aeb12af0df618f581d7.js"></script>

*Et voila!* Problem solved! Now the data is retained on configuration change and the dependent property is working as expected.

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/-VA9WSJl8Sk" frameborder="0" allowfullscreen></iframe></center>


**Bonus: LiveData**

Like ViewModel, [LiveData](https://developer.android.com/topic/libraries/architecture/livedata) is another class from the Lifecycle Architecture Component library. LiveData is a data holder which also happens to be lifecycle-aware, meaning it respects the lifecycle of other app components, such as activities, fragments, this ensures that if an activity or fragment is observing a LiveData, it will only send an update to them in an active lifecycle state.

Android Studio 3.1 brought another sweet update to the DataBinding library:
> You can now use a [LiveData](https://developer.android.com/reference/android/arch/lifecycle/LiveData.html) object as an observable field in data binding expressions. The [ViewDataBinding](https://developer.android.com/reference/android/databinding/ViewDataBinding.html)class now includes a new setLifecycleOwner() method that you use to observe [LiveData](https://developer.android.com/reference/android/arch/lifecycle/LiveData.html) objects.

So instead of using ObservableField’s I used LiveData for binding the data to the UI. The property dependency can be achieved by using a [MediatorLiveData](https://developer.android.com/reference/android/arch/lifecycle/MediatorLiveData): I created another top-level inline function only this time returning a MediatorLiveData instead of ObservableField and its dependencies were LiveData instead of Observable:

<script src="https://gist.github.com/ArthurNagy/2f517968379039d16a9181f8fac5f2cf.js"></script>

All that was left do is swapping out all **ObservableField** properties to **LiveData** in the **UserViewModel**:

<script src="https://gist.github.com/ArthurNagy/680ac11586f0322dd5b960c31753cb50.js"></script>

and calling **setLifecycleOwner()** on the binding instance in the activity:

<script src="https://gist.github.com/ArthurNagy/33388d19fae688275917473e778a8302.js"></script>

The end!

Hope you find this feature useful and now that you’ve seen how easy it is to integrate it into your codebase, you’ll start experimenting with it. I’d be thrilled to hear your thought on this functionality, as well as ideas about cases you see yourself using it.

You can find all the code [here](https://github.com/ArthurNagy/DataBindingPlayground). Any feedback, and shares are more than welcome.

Original article posted on medium [here](https://medium.com/halcyon-mobile/android-architecture-components-databinding-dependent-properties-6e8eba6c8b13).