---
layout: post
title:  "Lessons Learned from implementing a Fragment to Fragment Shared Element transition"
date:   2019-04-30
categories: Dev
tags: [reactive, reactor, kotlin]
---
|---|---|
|![Discover transition gif](/assets/fixed_transition_discover.gif)|![Recent issues transition gif](/assets/fixed_transition_recent_issues.gif)|

### 1. Fragments should be in the same container
You cannot do shared element transitions with fragments in another container.

### 2. Use `.replace()` instead of `.add()`
Shared element transitions do not work when adding fragments, even to the same container.

### 3. Enable reordering allowed when postponing transitions
When the content of the transition needs to be loaded before the transition can start, we need to use `postponeEnterTransition()`. Postponing the transition will only work when you add `setReorderingAllowed(true)` to your fragment transition (see [Android docs](https://developer.android.com/reference/android/support/v4/app/FragmentTransaction.html#setreorderingallowed) and the Reordering part of [this blog post by Chris Banes](https://chris.banes.dev/2018/02/18/fragmented-transitions/) for more context.)

### 4. Properly resume the state of the fragment you return to with the return transition
In this case, the view that needs to be resumed contains a nested `RecyclerView` within the main `RecyclerView`. The shared element needs to be in view in both fragments when the transition takes place. Therefore the nested `RecyclerView` needs to be restored to it's previous state before the transition is started. This is done automatically when we use `.add()` to add the view because that means the view will remain in memory. However, using `.replace()` means that the fragment to which we need to return has been destroyed and needs to be recreated through the fragment lifecycle.

Using the `onSavedInstanceState()` and `onRestoreInstanceState()` methods of the nested `RecyclerView`, the state needs to be manually restored. `startPostponedEnterTransition` should be called when the shared element on the restored fragment is fully rendered.

### 5. To transition a `CardView` with an image, add both the card and the image as shared elements
Doing the transition on just the `CardView` won't dynamically scale the image during the transition. If you do the transition on just the image, the card already be at its final position at the start of the transition.

### 6. `postponeEnterTransition()` and `startPostponedEnterTransition()` also work for return transitions
The naming of these functions is really confusing but they work exactly the same for return transitions.

### 7. `setCustomAnimations` fade-in/out causes shared element flicker. Individual fade-in/out fixes this
Using `setCustomAnimations(android.R.anim.fade_in, android.R.anim.fade_out)` to add the fragment fade-in/out animation causes a subtle flicker on the shared element whenever the shared element transition is finished. This is fixed by setting a separate fade-in/out transition on the individual fragments:

```kotlin
currentFragment.exitTransition = TransitionInflater.from(context).inflateTransition(android.R.transition.fade)
newFragment.enterTransition = TransitionInflater.from(context).inflateTransition(android.R.transition.fade)
```

However, these transitions persist past this fragment transaction. If the `exitTransition` of the fragment is not reset, a fade will be shown when returning to this fragment from other fragments, even if no specific animation is specified. We can easily clear this by resetting the `exitTransition` when the shared element transition is finished.

```kotlin
fragment.sharedElementEnterTransition = TransitionInflater.from(context).inflateTransition(android.R.transition.move).addListener(object : TransitionListenerAdapter() {
    override fun onTransitionEnd(transition: Transition) {
        // The current fragment transition should only be applied for this transition and be removed afterwards
        currentFragment.exitTransition = null
    }
})
```