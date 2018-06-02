---
title:  "Implementing Google’s refreshed modal bottom sheet"
date:   2018-05-22 15:00:00
categories: [android]
tags: [android]
---

This year’s Google I/O introduced an improved version of Material Design to the curious public in Mountain View and across the globe. With the new release a couple of components got a fresh coat of paint but some shiny new elements entered the game too. The tech giant debuted a few new and revamped apps that already reflect the changes in their official design language.

One of their new joiner apps is [Tasks](https://play.google.com/store/apps/details?id=com.google.android.apps.tasks&hl=en), a simple and clean productivity app that integrates well with other G suite apps. I’ve been using it since its release and I really like it.

[News](https://play.google.com/store/apps/details?id=com.google.android.apps.magazines), formerly known as Play Newsstand is another app that has been well received at the event. Besides its new Material theme News is now powered by AI.

Both of these products have a neat, simple and intuitive design, that showcases the capabilities of the updated [Material Design for Android](https://github.com/material-components/material-components-android). It’s stunning. So much so, that it got me taking these apps as design examples for aside project I started working on.

Shortly after I started implementing the UI, I stumbled upon my first issue: the BottomSheetDialog component, available through the [new design library](https://github.com/material-components/material-components-android/blob/master/docs/getting-started.md), did not look like the ones from theseapps:

![Left: Tasks, right: News, both apps have their menu open](https://cdn-images-1.medium.com/max/2800/1*rLBf6fPtfWhXqnTOLlj0cQ.png)*Left: Tasks, right: News, both apps have their menu open*

As you can see, both apps use a modal bottom sheet dialog to present their menu, but their top edges are rounded and the NavigationBar isn’t dimmed. If we use the BottomSheet from the MaterialComponents library this is what we get:

![Left: app with menu closed, on the right we have the menu open which brings up a modal bottom sheet](https://cdn-images-1.medium.com/max/2800/1*6BxLHKE97eBkGTeG12MT4w.png)*Left: app with menu closed, on the right we have the menu open which brings up a modal bottom sheet*

The screenshot from the left hand side shows the home screen with its menu closed. The one on the right illustrates how the modal bottom sheet is shown when the menu icon from the bottom app bar is tapped.

The question: how can one implement the same behaviour that’s present in the apps from the examples? My goal was to do it as simple and elegant as possible, so here’s how I did it:

First I created a shape drawable which I’ll be using as background for the bottom sheet:

<script src="https://gist.github.com/ArthurNagy/0ecff500a40508682e242f4243e72f18.js"></script>

Next, I made a custom style for the BottomSheet widget and a custom theme for the BottomSheetDialogFragment:

<script src="https://gist.github.com/ArthurNagy/cf6f355e7760de69e086662bf857fc8b.js"></script>

In the BottomSheet style, I set the custom background drawable to be used as the bottom sheet’s background. After that I created a custom theme forthe fragment, set the statusBarColor to transparent and the navigationBarColor to white (that’s our desired color in this case). Then I set the fragment’s window [floating attribute](https://developer.android.com/reference/android/view/Window#isfloating) which wraps the dialog to false.

Lastly, I extended the BottomSheetDialogFragment and set my new theme on it:

<script src="https://gist.github.com/ArthurNagy/aab0bf71f39c8e4666f22ff69d3a35c0.js"></script>

Aaaand that’s it. All that was left to do is extend the RoundedBottomSheetDialogFragment, inflate a layout of my choice in it and simply show it, like you’d do with a normal BottomSheetDialogFragment:

<script src="https://gist.github.com/ArthurNagy/85f5ebc5db8a5cc0cd715f391323c860.js"></script>

The result:

![Rounded BottomSheetDialog fragment without dimmed NavigationBar](https://cdn-images-1.medium.com/max/2000/1*8rSZqfchMsVbWTni0Drqxg.png)*Rounded BottomSheetDialog fragment without dimmed NavigationBar*

You can find all the code [here](https://gist.github.com/ArthurNagy/1c4a64e6c8a7ddfca58638a9453e4aed). Let me know if you have a similar or better solution. Any feedback, and of course shares and claps are more than welcome.

[Implementing Google’s refreshed modal bottom sheet](https://medium.com/halcyon-mobile/implementing-googles-refreshed-modal-bottom-sheet-4e76cb5de65b)