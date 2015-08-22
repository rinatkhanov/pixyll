---
layout:     post
title:      Custom UINavigationController transition animations with Swift
date:       2015-08-21 13:21:19
categories: ios
summary:    "In the world of flat design, animations play quite an important role – when done right, they guide users throughout the app, making it more delightful and enjoyable experience. Your app may function good enough with built-in system transitions, but custom animations often make a huge difference in overall feeling, making your work stand out as a piece of fine craftsmanship.
<br><br>
I am a big fan of animation libraries such as Facebook's pop and Spring because they do all the hard work under the hood making it super easy to create awesome animations in just a few lines of code. However, I've always had this impression that it is quite difficult to implement transitions from one screen to another in UINavigationController stack – I could animate stuff in viewDidAppear & viewWillDisappear occasionally, but it's icky and adds a lot of boilerplate code to view controllers. So most of the time I just sticked to built-in slide-from-right animation we're so familiar with.
<br><br>
I was aware of new view controller transitioning API introduced back in iOS 7 but until very recently, I never got around to actually take a look at it. Apparently, it is much easier than I imagined! In this article, I am going to make a simple introduction to custom UINavigationController transitions for those of us who are still not familiar with this API."
---

In the world of flat design, animations play quite an important role – when done right, they guide users throughout the app, making it more delightful and enjoyable experience. Your app may function good enough with built-in system transitions, but custom animations often make a huge difference in overall feeling, making your work stand out as a piece of fine craftsmanship. 

I am a big fan of animation libraries such as Facebook's [pop](https://github.com/facebook/pop) and [Spring](https://github.com/MengTo/Spring) because they do all the hard work under the hood making it super easy to create awesome animations in just a few lines of code. However, I've always had this impression that it is quite difficult to implement transitions from one screen to another in `UINavigationController` stack – I could animate stuff in `viewDidAppear` & `viewWillDisappear` occasionally, but it's icky and adds a lot of boilerplate code to view controllers. So most of the time I just sticked to built-in slide-from-right animation we're so familiar with.

I was aware of new view controller transitioning API introduced back in iOS 7 but until very recently, I never got around to actually take a look at it. Apparently, it is much easier than I imagined! In this article, I am going to make a simple introduction to custom `UINavigationController` transitions for those of us who are still not familiar with this API.

![transition-animation](https://cloud.githubusercontent.com/assets/4293969/9360515/8b3b49c0-468e-11e5-8af6-aa358f7b05a7.gif)

#### Basics

The first thing we need to do is create a delegate for our navigation controller. Let's make a new class conforming to `UINavigationControllerDelegate` protocol and implement one of the (optional) methods related to transition animations:

```swift
class NavDelegate: NSObject, UINavigationControllerDelegate {
	private let animator = Animator()

	func navigationController(navigationController: UINavigationController, 
         animationControllerForOperation operation: UINavigationControllerOperation, 
                         fromViewController fromVC: UIViewController, 
                             toViewController toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return animator
    }
}
```

If you want to have custom animation only for push or pop transition, you can return `nil` here instead of your animator object (`operation` argument indicates whether it's push or pop). In that case, system would fallback to standard sliding animation. Pretty neat!

There are different ways to set an instance of this class as a delegate for our `UINavigationController`, but I recently learned an Interface Builder trick which allows us to accomplish this without writing any code:

![delegate-storyboard](https://cloud.githubusercontent.com/assets/4293969/9360551/bd48283e-468e-11e5-807a-21dc3ab4d7cd.png)

You probably noticed `Animator` class in there, which as the name suggests handles all the animations. You could essentially have animation code in `NavDelegate` class itself and just return `self` in that method above, but I like to keep things separate (even though I define `Animator` in the same `.swift` file).

```swift
class Animator: NSObject, UIViewControllerAnimatedTransitioning {
    func transitionDuration(context: UIViewControllerContextTransitioning) -> NSTimeInterval {
        return 0.4
    }
    
    func animateTransition(context: UIViewControllerContextTransitioning) {
        let toVC = context.viewControllerForKey(viewControllerForKey:UITransitionContextToViewControllerKey)
        let fromVC = context.viewControllerForKey(viewControllerForKey:UITransitionContextFromViewControllerKey)
        context.containerView().addSubview(toVC.view)
        // animate your views here, then call this method when your animation is completed:
        context.completeTransition(!context.transitionWasCancelled())
    }
}
```

It conforms to `UIViewControllerAnimatedTransitioning` protocol and implements both of required methods. 

And that's it, really! You can get your view controllers from context object which is passed as an argument and animate views in `animateTransition` method. 

(A little side note: if you're wondering why both `NavDelegate` and `Animator` inherit from `NSObject`, it's because their protocols `UINavigationControllerDelegate` and `UIViewControllerAnimatedTransitioning` inherit from `NSObjectProtocol`).

Your animation may be as simple as plain cross-dissolve or scale transform, but it could also be anything arbitrary complex. I am going to spend the rest of this post explaining how to achieve animation showed on GIF above, but the general approach of implementing custom navigation controller transition animations with Swift should be pretty clear now.

#### Custom transition example

So, let's imagine we have a table view in one view controller (`CategoryVC`) with cell backgrounds acting as a hero image in another view controller (`PostListVC`) and we want to smoothly animate the transitioning between these two screens. Because `Animator` handles both push and pop transition in my case, let's change our implementation of `animateTransition` method to reflect that:

```swift
func animateTransition(context: UIViewControllerContextTransitioning) {
    // push
    if let categoryVC = context.viewControllerForKey(UITransitionContextFromViewControllerKey) as? CategoryVC,
        postListVC = context.viewControllerForKey(UITransitionContextToViewControllerKey) as? PostListVC {
            moveFromCategories(categoryVC, toPosts:postListVC, withContext: context)
    }
        
    // pop
    else if let categoryVC = context.viewControllerForKey(UITransitionContextToViewControllerKey) as? CategoryVC,
        postListVC = context.viewControllerForKey(UITransitionContextFromViewControllerKey) as? PostListVC {
            moveFromPosts(postListVC, toCategories: categoryVC, withContext: context)
    }
}
```

Cool, now we have two separate methods to handle push and pop transitions between our screens. Let's take a look at our animation once again, this time in slow motion: (you can enable it by selecting Debug -> Slow Animations in the simulator or hitting ⌘T)

[slowmo-transition]

As you can see, there's a lot happening during animation. Let's break it down into parts, focusing just on push transition for now:

```swift
private var selectedCellFrame: CGRect? = nil
private var originalTableViewY: CGFloat? = nil

private func moveFromCategories(categoryVC: CategoryVC, toPosts postListVC: PostListVC, withContext context: UIViewControllerContextTransitioning) {
            
    if let indexPath = categoryVC.tableView.indexPathForSelectedRow(),
        selectedCell = categoryVC.tableView.cellForRowAtIndexPath(indexPath) as? FolderCell {
        
            context.containerView().addSubview(postListVC.view)
            
            // cell background -> hero image view transition
            // (don't want to mess with actual views,
            // so creating a new image view just for transition)
            let imageView = createTransitionImageViewWithFrame(selectedCell.frame)
            imageView.image = selectedCell.background.image
            imageView.alpha = 0.0 // hidden initially
            postListVC.view.addSubview(imageView)

            // save table view's original position and selected cell frame
            // (as a property) to move them back during pop transition animation
            selectedCellFrame = selectedCell.frame
            originalTableViewY = categoryVC.tableView.frame.origin.y

            // figure out by how much need to move content
            let heroFinalHeight = PostListVC.HeroViewHeight.Regular.rawValue
            let deltaY = selectedCell.center.y - heroFinalHeight / 2.0

            // adjust text labels inside hero view (so they appear as if they came from selected cell)
            let originalCategoryDescriptionBottomSpacerConstant = postListVC.categoryDescriptionBottomSpacer.constant
            postListVC.categoryDescriptionBottomSpacer.constant -= deltaY / 2.0

            postListVC.hideElementsForPushTransition() // (more about that later)
            
            UIView.animateWithDuration(0.5, delay: 0.0, usingSpringWithDamping: 0.75, initialSpringVelocity: 1.0, options: .CurveEaseInOut, animations: {

            	// hide "your future pack" label
                categoryVC.titleLabel.alpha = 0.0

                // adjust table view frame so it appears like whole content is moving with cell image
                categoryVC.tableView.frame.origin.y -= deltaY

                // move our transitioning imageView towards hero image position (and grow its size at the same time)
                imageView.frame = CGRect(x: 0.0, y: 0.0, width: imageView.frame.width, height: heroFinalHeight)
                imageView.alpha = 1.0

                postListVC.view.alpha = 1.0
                
                }) { finished in
                    
                    // now we are ready to show real heroView on top of our imageView
                    postListVC.view.sendSubviewToBack(imageView)

                    postListVC.categoryDescriptionBottomSpacer.constant = originalCategoryDescriptionBottomSpacerConstant
                    postListVC.prepareToCompletePushTransition() // (more about that later)
                    
                    // prepare constraints for animation
                    let autoLayoutViews = [postListVC.backButton, postListVC.categoryDescription, postListVC.categoryTitle]
                    for view in autoLayoutViews { view.setNeedsUpdateConstraints() }
                    
                    UIView.animateWithDuration(0.3, animations: {
                        postListVC.heroView.alpha = 1.0
                        for view in autoLayoutViews { view.layoutIfNeeded() }
                        
                        }) { finishedInner in
                            
                            // clean up & revert all the temporary things
                            imageView.removeFromSuperview()
                            categoryVC.titleLabel.alpha = 1.0
                            categoryVC.tableView.deselectRowAtIndexPath(indexPath, animated: false)
                            
                            context.completeTransition(!context.transitionWasCancelled())
                    }
            }
    }
}
```

Bloody hell, that's a lot of code! Luckily, most of it is really self-explanatory adjusting of views. You may need to take a minute to follow along comments, but it's not a rocket science.

I want to point out that if you want to change layout of views with Auto Layout enabled, you'll need to adjust `constant` property of your constraints. As you probably already know, you can't change `frame` property directly for such views. And in order to animate these constraint changes, you need to call `setNeedsUpdateConstraints` outside of and `layoutIfNeeded` inside of `UIView` animation block.

Also, you may need to take into account `tableView.contentInset.top` property when getting selected cell's frame. 

There are a few helpers I omitted from code snippet above. For the sake of completion, let's take a look at them too:

```swift
private func createTransitionImageViewWithFrame(frame: CGRect) -> UIImageView {
    let imageView = UIImageView(frame: frame)
    imageView.contentMode = .ScaleAspectFill
    imageView.setupDefaultTopInnerShadow()
    imageView.clipsToBounds = true
    return imageView
}
```

Nothing particularly exciting here, just a plain `UIImageView` which is used for cell / hero view transition.

```swift
private extension PostListVC {
    func hideElementsForPushTransition() {
        // hero view appears with slight delay (not in sync)
        // so need to hide it explicitly from container view
        view.alpha = 0.0
        heroView.alpha = 0.0
        
        // hide all visible cells
        for cell in visibleCellViews { cell.alpha = 0.0 }
        
        // move back button arrow beyond screen
        backButtonHorizontalSpacer.constant = -70.0
    }
    
    func prepareToCompletePushTransition() {
        backButtonHorizontalSpacer.constant = 0.0
        disableTransparencyAnimatedForViews(visibleCellViews)
    }
    
    private var visibleCellViews: [UIView] {
        return (tableView.visibleCells() as! [UITableViewCell]).map { $0.contentView }
    }
}
```

Interesting part is `disableTransparencyAnimatedForViews` call which allows us to show cells one by one. It is implemented as a simple recursive function:

```swift
func disableTransparencyAnimatedForViews(views: [UIView]) {
    if let view = views.first {
        UIView.animateWithDuration(0.2, animations: { view.alpha = 1.0 }) { _ in
            disableTransparencyAnimatedForViews(Array(views[1..<views.count]))
        }
    }
}
```

You could achieve the same effect by overlaying semitransparent view on top of table view and sliding it towards bottom of the screen, but I like this approach more – even though it requires you to get rid of native table view separators and instead add thin line to the bottom of cell's content view. Whatevs.

Let's take a look at pop transition animation:

```swift
private func moveFromPosts(postListVC: PostListVC, toCategories categoryVC: CategoryVC, withContext context: UIViewControllerContextTransitioning) {

    context.containerView().addSubview(categoryVC.view)
    categoryVC.view.alpha = 0.0
    
    let imageView = createTransitionImageViewWithFrame(postListVC.heroView.frame)
    imageView.image = postListVC.categoryHeroImage.image
    context.containerView().addSubview(imageView)
    
    UIView.animateWithDuration(0.4, animations: {
        postListVC.view.alpha = 0.0
        postListVC.view.transform = CGAffineTransformMakeScale(0.9, 0.9)
        categoryVC.view.alpha = 1.0
        categoryVC.tableView.frame.origin.y = self.originalTableViewY ?? categoryVC.tableView.frame.origin.y
        imageView.alpha = 0.0
        imageView.frame = self.selectedCellFrame ?? imageView.frame
    }) { finished in
        postListVC.view.transform = CGAffineTransformIdentity
        imageView.removeFromSuperview()
        context.completeTransition(!context.transitionWasCancelled())
    }
}
```

Basically, it's the same thing as push transition, just in reverse order and much simpler in my case (less moving parts).

Notice how you can scale your views during transition by setting `transform` property (don't forget to reset it to `CGAffineTransformIdentity` in completion).

As you can see, it's quite a lot of code, but most of it is preparing and setting up views for transition, and it really depends on how advanced animation is that you want to create. 

I also want to note that it is possible to make custom navigation controller transitions interactive – similar to how swipe-to-go-back gesture works on iOS. This feature is out of scope for this post, but you may want to read [Chris Eidhof's article](http://www.objc.io/issues/5-ios7/view-controller-transitions/) for more information about that.

Thanks [Sal](https://twitter.com/khanov) and [Radek](https://twitter.com/radexp/) for reading drafts of this post.