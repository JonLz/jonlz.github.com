---
title: "GIF Memory Game"
date: 2016-02-20T22:40:49-05:00
draft: false
---

# Overview

Welcome, if you haven't already please check out the [GitHub repo](http://www.github.com/JonLz/gif-memory-game) and have it open (or the project built and open) as you read along.This guide is intended to be a companion post that will walk through each commit and discuss the development process.

### What is this?

An animated GIF memory game iOS app. The GIFs will be pulled from Giphy with some user customization based on trends provided by Twitter. We'll be rolling everything else ourselve with some helpful cocoapods.

### Who is this for?

Any aspiring iOS developer trying to learn Objective-C who would like a small, simple, fully built project to poke around and explore. This is aimed at junior/pre-junior level - I doubt anyone beyond that will take away much. You will need to know some Objective-C and be familiar with the language works. The intent isn't to explain how every line of code works - this will be a high level overview of how you can use some of the pieces. If you find any particular pieces interesting, please explore them further in more depth.

### Why should you care?

Animated GIFs are awesome. Learning iOS is awesome.

You want see the following in action in Objective-C: 

+ Networking
+ Table Views
+ View Controller Transitions
+ Custom UIViews
+ Modeling Data
+ Persisting Data (NSKeyedArchiver/NSKeyedUnarchiver)
+ Key-Value Observing
+ Views in Code
+ Autolayout
+ Commonly used Pods
  + AFNetworking
  + SDWebImage
  + Masonry
  + OHHTTPStubs

### How are we going to do it?

This project will be completely in Objective-C. Swift is cool and all, but you should know some Objective-C. It's not going anywhere anytime soon. You'll need at least iOS 9, XCode 7, and Cocoapods installed. You don't need git installed but it's helpful if you want to track along with each commit as the project is built. I won't be covering how to get your IDE setup as that is outside the scope of this series. Check out the github repo for notes on how to build the project. Let's get started!

### How to follow along

I recommend cloning the project locally and then checking out (git checkout) each commit as it's made. For each commit you can do 'git hist' (while in the master branch) and then 'git checkout <X>' where <X> is the commit hash (the random 7 digit alphanumeric string). That will let you poke around that commit as a branch and hide all the future commits made. If you want to build the project, you'll have to install the pods (pod install) first. When you're done, just 'git checkout master' to go back and checkout the next commit.

---

### Abstractions

Now that we've nailed down the scope, let's think abstractly about what we'll need to accomplish. 

If we take a step back and think about the game of memory, what is it? It's just a set of tiles, cards, or pictures, with an associated ruleset. I'll use the working concept of tiles since that's what I chose when naming it in this project. Each tile has two states, visible or not visible. If you flip over two of the same tile, they both stay visible. If they're different, they get flipped back over. You repeat this process until all the tiles are flipped over. 

We're going to need to have some way of keeping track of the game (number of turns, selected tiles, matched tiles, etc.). This will be what you sometimes hear referred to as 'business logic' - it's quite literally the guts of the game.

We also will have to take that tracked game data and present it on screen to the user so they know what tiles are available / selected. We'll need to interact with their touches to further the game progress. 

We should also save game progress in case the user closes the app.

We'll also need to pull in images from a third party API and use them as part of our game. Pretty simple. 

Finally we will need to think about how to provide some customization options to make the  game a little more engaging.

Let's do this.

---

### Commit 1 - 3 - Project Setup

Nothing interesting here. We are just setting up the project, adding podfiles, and assets. So far for pods I've decided to use AFNetworking, Masonry, and SDWebImage. I'll talk more about these later but for now here is a quick overview:

AFNetworking is a nice wrapper around the core networking libraries. It handles a lot of the nuisance of working with networking code including having sane defaults, built in request serializers, and handling callbacks on the main thread. 

Masonry is a wonderful DSL for autolayout that makes it a lot more expressive and manageable.

SDWebImage handles loading images from the web directly into UIImageViews as well as caching those images and a whole lot more.

I encourage checking out each of the repos for these to read the documentation and understand how they work at a high level.

---

### Commit 4 - Title Screen

Now we have a pretty basic title screen view controller that will display a background, a title, and show a menu. The user can tap on the menu which doesn't do anything yet but we've setup the gesture recognizers to listen for those events.

#### Autolayout

There's a litany of things I could say about autolayout but suffice it to say it's worth learning. The basics of autolayout are that it is a constraint solving system. It uses a bunch of algorithms and linear equation solving to generate frames for you based on where you tell it you want them to be. It's not perfect and its certainly not a solution to be used everywhere, but even for simple layouts I find it easier to use than setting up custom frames and CGRects for every element. Especially so since I like to do all my views programatically.

Masonry comes in and simplifies the way you interact with autolayout to a point where you are just telling the autolayout system exactly what you want done in a concise manner. Masonry isn't magic - it's harnessing the power of blocks (that's what that 'make' is) to take your inputs and send them into the autolayout system. The nice thing about Masonry is when your constraints overlap (the system can't solve two constraints simultaneously due to the way you set it up) it will do its best to tell you which views those are and some direction on where to investigate.

Just a couple of quick notes here. For elements that use autolayout, do not use frames to try to interact with that element - there is a way around this but for now we can ignore that. Autolayout will infer a frame for you based on the constraint and if you try to override that frame, it can conflict and you will likely receive NSAutoresizingMaskLayoutConstraint complaints. Not fun. This is especially true for simple layouts like we're doing here.

I like to setup my layouts from the top down (for vertical views). I start with the topmost element and constrain it to where I want it, then work down and setup views relative to the top views so everything hangs down from the top. Here we are setting a background image to be the entire size of the screen (self.view refers to the base view of the view controller). Then we tell the title label it should be 90 points below the top of the screen. Then we center it horizontally. Finally, the menu stack view should have its bottom 50 points from the bottom of the screen, be as wide as the screen, and have a height of 150.

```objc
[self.backgroundImageView mas_makeConstraints:^(MASConstraintMaker *make) {
	make.edges.equalTo(self.view);
}];
 +
[self.titleLabel mas_makeConstraints:^(MASConstraintMaker *make) {
    make.top.equalTo(self.view.mas_topMargin).mas_offset(@90);
    make.centerX.equalTo(self.view);
}];

[self.menuStackView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.bottom.equalTo(self.view.mas_bottomMargin).offset(-50);
    make.left.and.right.equalTo(self.view);
    make.height.equalTo(@150);
}];

```

#### Intrinsic Content Size

You may be asking why we didn't use autolayout to set the size of each of the menu labels. This is because some views have an intrinsic content size that is based on the content of the view. This is true of UILabels where they can infer their size based on a combination of the font, font size, number of lines, etc. When this is the case, we don't need to explicitly set the frame (via autolayout). So when using views with intrinsic content size we don't need to worry about the width and height constraint, just the x and y constraint so autolayout knows where to put it. Although in our case, we get to ignore the x and y constraint as well because the stack view will handle that for us.

Here's further [reading material](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/ViewswithIntrinsicContentSize.html) on autolayout / intrinsic content size. 

#### Stack Views

Stack views are a great new tool provided in iOS 9.0+. They help manage content that is arranged in a stack either horizontally or vertically. The content doesn't all have to be the same size although in our case it is. The beauty comes when you combine stack views to easily create relatively complex views. You can have rows of horizontal stack views all in one vertical stack view to easily create a nice grid. This is what we'll use later for the game screen. For now we've just chosen a simple vertical stack view to represent our menu.

One thing to keep in mind is that stack views use autolayout to layout their contents.

```objc
self.menuStackView.alignment = UIStackViewAlignmentCenter;
self.menuStackView.distribution = UIStackViewDistributionFillEqually;
self.menuStackView.axis = UILayoutConstraintAxisVertical;
self.menuStackView.spacing = 8.0f;
[self.menuStackView addArrangedSubview:self.startLabel];
[self.menuStackView addArrangedSubview:self.aboutLabel];
```

You can read the [documentation](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIStackView_Class_Reference/) on what each of these properties does but very quickly, we have a stack view on the vertical axis, that aligns elements to the center, fills elements equally, and provides 8 points of spacing.

---

### Commit 5 - 7 - Networking

There's quite a bit going on here, but I'm going to assume that you understand how REST APIs work because that is pretty crucial to understanding the networking code. If you don't, please read up on REST, APIs, JSON, HTTP - the protocol itself and the verbs it uses - before continuing. I'll do my best to explain it but it's quite a tall order as I feel like I'll be explaining The Internet in 2 paragraphs.

#### REST

REST (REpresentational State Transfer) is an architectural style wherein a service/server can provide agreed upon endpoints for consumers to consume a resource. A resource is typically a JSON response. JSON stands for javascript object notation and it's what a large majority of web services use to talk to each other. It's just a collection of arrays and dictionaries which we'll see later. 

So if someone provides an endpoint, how do we interact with it? Turns out there are a bunch of ways and the HTTP protocol (redundant I know) specifies exactly how to do this. Some of the more common interactions include POST, GET, PUT, DELETE. This is commonly referred to as the CRUD operations - create, read, update, delete. We'll be reading a resource below.

Once a server receives a request, it responds to it with a response which includes a status code. Two of the codes you should know are 200 - OK and 404 - Not found. There are a lot of [others](http://www.restapitutorial.com/httpstatuscodes.html) which you can browse.

Our goal here is to send a GET request to a REST endpoint over at [Giphy](https://github.com/Giphy/GiphyAPI) (/gifs/trending), receive back the JSON response and parse it.

```objc
 +(void)retrieveGifUrlsWithSuccess:(void (^)(NSArray *urls))success failure:(void (^)(NSError *))failure
 {
     AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
     
     NSString *url = @"http://api.giphy.com/v1/gifs/trending?api_key=dc6zaTOxFJmzC&limit=24";
     
     [manager GET:url parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
         
         NSArray *results = responseObject[@"data"];
         NSMutableArray *urls = [[NSMutableArray alloc] init];
         
         for (NSDictionary *result in results) {
             NSString *fixedHeightImageURL = result[@"images"][@"fixed_height"][@"url"];
             [urls addObject:[NSURL URLWithString:fixedHeightImageURL]];
         }
         
         success(urls);
         
     } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
         failure(error);
         
     }];
 }
```

#### AFNetworking

Here you can read about the [AFHTTPSessionManager](http://cocoadocs.org/docsets/AFNetworking/3.0.4/Classes/AFHTTPSessionManager.html) class

In the first couple lines we create a new manager, setup the URL we want to access, then tell AFNetworking to go GET the resource. It will go do its thing and then return back with a block callback that is either successful or failed. Behind the scenes it will parse the response code from the server and infer whether it was successful or not. 

If successful, it returns back a responseObject which is a JSON response from Giphy that it has already deserialized into an NSDictionary or NSArray (based on the JSON structure). Tke a look at GiphyResponse.json file to see what the response looks like. There are a lot of JSON viewers available online. Also a good tool that I like to use is Postman which is a chrome extension for sending requests and seeing the response. You can paste in the URL above and see what you get back in raw format or JSON.

Once we have the responseObject back, a quick look into it shows that it's an NSDictionary. It contains a data key which contains an array of dictionaries. We can iterate through each of those dictionaries and pull out only what we want. In our case, it's the URL field which is buried a few dictionaries deeper.

Here's a screenshot of what the JSON response looks like:

![Giphy JSON Response](/projects/assets/GiphyJson.png)

Once we build up our array of URLs, we provide that back to the caller via a completion block and we're done.

#### OHHTTPStubs

Very briefly, OHHTTPStubs is a framework that lets you provide stubbed HTTP responses for network calls within your app. This is a great tool so you don't need to send a network request every time you test your app. While a lot of APIs these days aren't ratelimited to the point that this might be an issue it's still nice to do and know how to do. You can also specify a specific JSON response for testing purposes too.

One thing worth mentioning is this is usually used with regards to testing but we haven't done that here yet. This is just for quick testing and I really wanted to stub the network response instead of sending one out every time I ran the code. Usually you would write a unit test of some sort that turned on the stubs and provided a stubbed response. Here we've just done that in our networking model instead of our test. By the way, a stub is just what it sounds like. Instead of going out to the internet to get a full response from a server, we've intercepted the request and responded back with our own. You'll hear the concept of stubs and mocks a lot when dealing with testing - here we've just used stubbing (providing a canned response). We won't get into mocking here but it's worth reading about on your own.

Instead of the server response, we've provided our own file (which I got by using Postman from sending a request and copying the response) in GiphyResponse.json

```objc
NSBundle *bundle = [NSBundle bundleForClass:[self class]];
[OHHTTPStubs stubRequestsPassingTest:^BOOL(NSURLRequest *request) {
	return YES;
} withStubResponse:^OHHTTPStubsResponse * _Nonnull(NSURLRequest * _Nonnull request) {
	return [OHHTTPStubsResponse responseWithFileAtPath:[bundle pathForResource:@"GiphyResponse" ofType:@"json"] statusCode:200 headers:@{@"Content-Type":@"application/json"}];
	}];
}
```

The first block here just returns YES so that it will stub every network request. Eventually we'll need to change this so it's just for the specific network call that we want because right now every network request we make is going to respond back with the Giphy JSON. You can do this by looking at the request URL and deciding whether you want to stub it or not. The pod website suggests this:

```objc
return [request.URL.host isEqualToString:@"mywebservice.com"];
```

The second block accesses the JSON response we've created ourselves and sends it back to whatever method requested the network request.

#### Singletons

```objc
+(instancetype)sharedInstance
 {
     static dispatch_once_t once;
     static id sharedInstance;
     dispatch_once(&once, ^{
         sharedInstance = [[self alloc] init];
     });
     return sharedInstance;
 }
```

The last piece of the puzzle here is this bit of code which is one way to setup the singleton pattern. This is a fairly common pattern you'll see throughout iOS development. What it intends to do is provide a single instance of one class that anyone can access by calling the sharedInstance class method. the static and dispatch_once code is the standard way to set this up so that an instance is only created once and then provided back to anyone who asks for the shared instance. 

This pattern can be nice if you want to only ever have one instance of a class and still make use of instance properties. Although we aren't using instance variables here this is still what we want since we only want one object in charge of performing all the network requests. It wouldn't make sense to be spawning 10 seperate objects that are all doing networking requests in our situation.

As an aside, you may also see this used in places to act as a global data store in order to pass data around between objects. There is much debate over whether this is a good way to use singletons or not and it can certainly be easy to have one place to manage global state, if you aren't careful it can often get the best of you. Please read [this article](https://www.objc.io/issues/13-architecture/singletons/) and then form your own opinions.


---

### Commit 8 - Game Model

Here we have the first iteration of the data models to model our memory game including variables to track state and logic to handle the game turns. I won't go into it too much but I've decided to use a MemoryGame and MemoryTile object to represent our game. Hopefully the interface is straightforward enough to follow without additional explanation.

The MemoryTile object will keep track the URLs required to load the animated GIF and the ID so we can compare whether two tiles are equal. Eventually we'll add a little bit more here but for now this suffices. The MemoryGame object will keep track of game state with flags and variables such as number of turns, all of the game tiles, and all of the selected tiles, amongst other things.

One change which bears mentioning is the initWithDictionary initializer method in MemoryTile.We've moved all of the dictionary processing code over to this object as that's where it belongs. The networking model shouldn't care about how to process and serialize the network response into a tile object. That should be the job of the memory tile. In case the requirements change, the tile has full control over any necessary changes. Our network model should only care about getting a response and providing it back to the requester. 

Our MemoryGame now is all set to go. It can ask the networking model to go fetch some GIF data, take that data and tell the memory tile to make a new tile with that data. Now the game only has to care about game related concerns: how to make a new game, handle a turn, etc. All of the game logic is handled in handleTurn method. The game will accept a tile (this corresponds to the tile that was touched) and figure out what to do next. This will be changing in a few commits down the road but for now we're golden.

---

### Commit 9 - Views and Interactions

Now that we've built our game model and can grab data from the internet to populate it, we'll work towards presenting it on screen and let the user interact with it. We'll make some custom UI elements to represent the tiles as well as a custom background view just for fun. We'll wrap that all up inside a game view controller whose job it will be to manage those individual UIViews and talk to the MemoryGame model. This will set us up nicely within the parameters of MVC architecture - we have a Model to keep track of state and logic, views that present information on screen, and a Controller to dictate the flow of data back and forth between the Model and View.

#### Core Graphics

I took the code for the background view directly from [here](http://www.raywenderlich.com/33496/core-graphics-tutorial-patterns). Read the tutorial there if you're interested. What this code does is use the Core Graphics engine to perform custom drawing of our tile background. I could've just grabbed an image and used it but it's nice to know the power of the iOS graphics libraries (plus trig is always fun, right?).

One thing to mention here is that because the final drawing code is placed in the UIView drawRect method, that is the code that will be used to render the view. We don't need to do anything except create the view and tell it where to go. drawRect handles the actual custom drawing. We implicitly tell it what size to be with autolayout - and it should be sized to whatever size the memory tile view is. The memory tile view will be whatever size the stack view tells it to be. And the stackview will be however big the view controller tells it to be.

#### UIControl

I've chosen to use a UIControl for the MemoryTileView instead of a UIView. Per [Apple](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIControl_Class/), "UIControl is the base class for control objects such as buttons and sliders that convey user intent to the application. You cannot use the UIControl class directly to instantiate controls. It instead defines the common interface and behavioral structure for all its subclasseses"

The intent of this subclass is to be a single unit of clickable content that the user can interact with and we would like to do something each time a tile is clicked. This is the perfect case to use a UIControl. We could have used a UIView and a gesture recognizer and that would be perfectly valid but you'll see why UIControl is easier to use for this situation shortly.

By the way, UIButton inherits from UIControl so you can liken what we're doing to making a custom UIButton (although this isn't strictly true). I won't get into why we don't subclass UIButton for this use case except that UIControl is a more lightweight approach. You certainly can subclass UIButton and this might be a good opportunity to read the docs on UIControl and UIButton to find out just how Apple constructed UIButton from UIControl.

#### Target/Action

UIControl makes available the Target/Action pattern which we'll make use of to listen for touch events on our memory tile views. You can read more about it [here](https://developer.apple.com/library/ios/documentation/General/Conceptual/Devpedia-CocoaApp/TargetAction.html). This pattern lets us respond to taps by binding the GameViewController (self) as the target for the action UIControlEventTouchUpInside. This is an action that is fired whenever the user touches inside of the memory tile view. This would also be a good time read about [Event Delivery and The Responder Chain](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html)

```objc
 for (MemoryTileView *tileView in self.memoryTileViews) {
         [tileView addTarget:self action:@selector(memoryTileViewTapped:) forControlEvents:UIControlEventTouchUpInside];
     }
```

#### SDWebImage

SDWebImage is a nice library that lets us point a URL at a UIImageView and it will handle the loading and caching for us. There isn't much to say about it except that it provides a lot of nice guarantees. With it, we can be safe in the assumptions that the downloading and displaying won't block the main UI thread, will be cached so multiple calls to the image wont be downloaded multiple times, and that it does this pretty efficiently. Oh and it also handles animated GIFs which isn't something that is native to iOS.

```objc
[_imageView sd_setImageWithURL:tile.smallAnimatedURL];
```

Pretty simple API isn't it? Yep, that's all we need to load an animated GIF from the internet.

#### Setting up the Views on Screen

With all the pieces in place, all we have to do now is display the memory tile views on screen. We do that with a healthy dose of stack views and a little autolayout. We'll iterate through our game tiles array, grab each tile, associate it with a memory tile view, and then display that view on the screen in a stack view. That's it.

```objc
+- (void)setupStackViews
{
	UIStackView *verticalStackView = [self verticalStackView];
  	for (NSUInteger i=0;i<6;i++) {
        UIStackView *horizontalStackView = [self horizontalStackView];
        for (NSUInteger j=0;j<4;j++) {
            NSUInteger index = i*4 + j;
            MemoryTile *tile = self.game.tiles[index];
            MemoryTileView *tileView = [[MemoryTileView alloc] initWithTile:tile];
            [horizontalStackView addArrangedSubview:tileView];
            [self.memoryTileViews addObject:tileView];
        }
        [verticalStackView addArrangedSubview:horizontalStackView];
    }
    
    [self.view addSubview:verticalStackView];
    [verticalStackView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.and.top.equalTo(self.view).mas_offset(@10);
        make.right.equalTo(self.view).mas_offset(@-10);
        make.height.equalTo(self.view).multipliedBy(0.75);
    }];
}
```

---

### Commit 10 - Syncing the Model and View

A couple of minor changes were made in this commit in order to clean up the game turn logic and as well let the game keep of track of when it will be ready to accept new touch events. This was achieved via the readyForNextTurn flag. When it's off, the game model will ignore any tiles passed into the handleTurn method. Recall that the view controller sends these events to the model every time the view is tapped. So now the user can tap as many times as they want but we'll only respond to the first two tiles tapped. 

#### Grand Central Dispatch

Grand Central Dispatch, aka GCD, is Apple's solution to managing complex multi-core computing environments in order to support concurrent code execution. What we typically use it for is to specify tasks to be run at certain times and/or on certain threads. Here we're using it to tell the system to run this code block 1 second after it's called and to run it on the main thread. This is a pretty standard snippet which you will see all over the place, so you should get used to reading it. 

We're doing this so that the user has time to process the two tiles before they disappear. If we didn't do this, as soon as the second tile is tapped both would instantly flip over and the game would be impossibly hard. Once we've waited 1 second, we flip the tiles over and update the game model to be ready to accept new touches.

dispatch_get_main_queue() makes sure it runs on the main thread. If you take away only one thing from reading this, and hopefully more than that since you've made it this far!, it's that the main thread is where all of the UI work must happen. Anytime you're doing anything that manipulates the UI, that is fiddling with UIViews, changing layer colors, moving things around, etc. do it on the main thread. We do this on the main thread because we'll be reading from these variables directly to alter our UI state later.

```objc
  dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            firstTile.visible = NO;
            secondTile.visible = NO;
            self.readyForNextTurn = YES;
        });
```

#### Key Value Observing

Key Value Observation is another common iOS pattern in which you can register an object to be an observer for some key somewhere else. This sets up a flow wherein when the key changes, the observer that registered to receive that change notification, will be notified and can respond appropriately. This is commonly used by views to receive updates from models in order to update the views.

One caveat here is that we have to make sure we deregister the observer in our dealloc method or else the app will crash once the key changes again and the observer is no longer around.

You might be wondering now why when I mentioned above that in MVC architecture it is the controller's responsibility to dictate the flow of data between the Model and the View. Well it certainly seems that is the view and model talking to each other directly without the controller's knowledge. This is true in a sense but MVC comes in many forms. 

In some architectures the view will send all events to the controller, the controller will do some work, talk to the model, and then respond back to the view. Here, the controller is involved because it has provided the model object to the view so that the view can do what it has to in order to update itself. The controller has done its job and because our use case is so simple and there is no inherent data transformation or complex request processing, we don't need to involve the controller in every interaction.

```objc
- (void)setupTileObserver
{
    [self.tile addObserver:self forKeyPath:@"visible" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context
{
    if ([keyPath isEqualToString:@"visible"]) {
        BOOL newValue = [[change valueForKey:NSKeyValueChangeNewKey] boolValue];
        BOOL oldValue = [[change valueForKey:NSKeyValueChangeOldKey] boolValue];
        
        self.imageView.hidden = oldValue;
        self.backgroundImageView.hidden = newValue;
        
    }
}
```

---

### Commit 11 - Cleanup

Nothing to add here. I removed storyboards and made sure the AppDelegate loaded the TitleViewController on app launch instead of loading the main storyboard.

---

### Commit 12 - Animations and Transitions

#### UIViewController Transitions

One recent (iOS 7+) feature added in to the iOS framework that I really like is the view controller transition API. It lets you create a transition object that will handle transitioning 2 different view controller views. You can be very specific about how to do it and it's not too hard to use once you understand how it works.  We'll use this to setup some basic view controller transitions to move one view off screen and bring another one on screen. I prefer this because the built in view controller transitions aren't well suited for games (i.e. modals/dismiss, push/pop) and don't give the user the sense that they are moving into another part of the game. Those transitions make sense for apps that display information or have a natural flow where they want to push and pop view controllers.

What we're trying to build here is a transition where a view slides off screen to the top and another one appears from the bottom at the same time. There are a couple of moving pieces here: the transition animator (an object), the from view controller, and the to view controller. 

The animator object is given a reference to each of the view controllers and it adds the views of those view controllers to a transition context (think of it like a temporary blank viewcontroller), handles the animation and then its job is done. What we need to do then is to setup the initial frame of the to view controller so that it starts in the correct place. For us, that frame will be one that is the same size as the main screen but one full height below the main screen (offscreen) so that the top of the to view controller is touching the bottom of the from view controller.Remember that we already have the initial frame of the from view controller because it's already on screen where we want it. So we'll tell the animator how to accomplish the animation we want and let it animate it.

Once we've setup the initial frames, we just use a UIView animation block to animate both views then tell the transition context to finish and we're done. We reverse this process to achieve an animation going the opposite direction which we will use when we dismiss the view controller (going back from the game to the title screen). Phew - one last topic and we'll get to the code.

The view controller transition API is setup with the use of delegates. The from view controller will act as the delegate for the to view controller. The delegate methods (shown below) tell each view controller what animator to use (the animator we created) and also whether we're presenting a view (i.e. a transition from title->game which is to push a view up off the screen) or dismissing a view (i.e. a transition from game->title where we'll push a view down off the screen). All we need to do in the delegate methods is create an animator, tell it whether its presenting, and then return the animator.

From the viewpoint of the from view controller (the view controller beginning the transition - here the title view controller) here's what needs to happen:

First we'll setup our from view controller to adhere to the protocol:

```objc
interface TitleViewController () <UIViewControllerTransitioningDelegate>
```

Then we need to implement the protocol methods:

```objc
- (id<UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented
                                                                  presentingController:(UIViewController *)presenting
                                                                      sourceController:(UIViewController *)source {
    
    TransitionAnimator *animator = [TransitionAnimator new];
    animator.presenting = YES;
    return animator;
}

- (id<UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed {
    TransitionAnimator *animator = [TransitionAnimator new];
    return animator;
}
```

And finally we can create a transition like we normally would:

```objc
- (void)startTapped
{
    GameViewController *gvc = [[GameViewController alloc] init];
    gvc.transitioningDelegate = self;
     
    [self presentViewController:gvc animated:YES completion:nil];
}
```

I will leave out the code from the actual animator object because it's easier for you to read through as its pretty straightforward frame and animation code.  Just remember the flow as you read through it from top to bottom. Grab view controllers, place their views in a temporary context, setup both view controller frames, do the animation, finish the transition.

Lots of good information [here](https://www.objc.io/issues/12-animations/custom-container-view-controller-transitions/) and [here](http://www.teehanlax.com/blog/custom-uiviewcontroller-transitions/) in much further detail.

#### UIView Transitions

I added in a basic flipping animation using a UIView transition that will show the user a tile flip animation whenever they tap the tile. This is using the built in UIView animation API that you can call on any UIView. We pass in some options that tell it to transition as a flip from the right, and to show/hide the views based on what should be displayed (the default behavior is to remove/add them from the view hierarchy which is not what I wanted). I've talked a little bit about UIView animations [here](https://medium.com/@jon.lazar/animations-and-cggeometry-in-ios-3374dfc57e37#.3br26xvt5), this method is similar although explicitly for transitioning between views and not for animating one view.

```objc
if (shouldBeVisible) {
    [UIView transitionFromView:self.backgroundImageView toView:self.imageView duration:0.5f options: UIViewAnimationOptionTransitionFlipFromRight | UIViewAnimationOptionShowHideTransitionViews completion:nil];
} else {
     [UIView transitionFromView:self.imageView  toView:self.backgroundImageView duration:0.5f options:UIViewAnimationOptionTransitionFlipFromLeft | UIViewAnimationOptionShowHideTransitionViews completion:nil];
}
```

Lastly, I added in a status box view to display the number of turns. It uses a combination of all the techniques we've discussed previously so there's nothing new to talk about. Check it out and see if you can understand how it works!

---

### Commit 13 - Persistence

Persistence in iOS is a pretty large topic and there are many different ways to achieve the goal of persisting app data to the disk. What option you choose will boil down to the tradeoffs, like pretty much everything in software engineering. You'll hear about core data, NSCoding/NSKeyedArchiver and Unarchiver, NSUserDefaults, 3rd party platform as a services, cloud solutions, sqlite, etc, etc. I'm not interested in getting into a debate about which is the best solution as I don't think that answer exists but there is a best solution for each problem. I encourage you to read more about the different options available in iOS, especially Core Data as that is widely used and is an excellent framework. However, for our use case our needs are incredibly simplistic. We'd like to persist our game object, game tile objects, and perhaps some user settings to the disk so they can close the app and not lose any progress.

#### NSKeyedArchiver / NSKeyedUnarchiver (NSCoding)
Enter NSKeyedArchiver/NSKeyedUnarchiver - possibly the easiest solution to persistence. With it, we get a simple interface to persist our objects but we lose any kind of easy access to relational querying, performance, etc. All we care about is that it's quick and easy to setup and it can persist our data. We don't need to query our data, care about how fast it loads, or do any kind of complex migrations. All that we need to do is to implement the NSCoding protocol which means you need to tell it how to map each of your object properties when it serializes and deserializes data. Let's take a look at how we do this for the MemoryGame object:

```objc
- (id)initWithCoder:(NSCoder *)aDecoder
{
    self = [super init];
    if (!self) return nil;
    
    _tiles = [aDecoder decodeObjectForKey:@"tiles"];
    _selectedTiles = [aDecoder decodeObjectForKey:@"selectedTiles"];
    _numberOfTurns = [aDecoder decodeIntegerForKey:@"numberOfTurns"];
    _readyForNextTurn = [aDecoder decodeBoolForKey:@"readyForNextTurn"];
    _gameInProgress = [aDecoder decodeBoolForKey:@"gameInProgress"];
    
    return self;
}

- (void)encodeWithCoder:(NSCoder *)aCoder
{
    [aCoder encodeObject:self.tiles forKey:@"tiles"];
    [aCoder encodeObject:self.selectedTiles forKey:@"selectedTiles"];
    [aCoder encodeInteger:self.numberOfTurns forKey:@"numberOfTurns"];
    [aCoder encodeBool:self.readyForNextTurn forKey:@"readyForNextTurn"];
    [aCoder encodeBool:self.gameInProgress forKey:@"gameInProgress"];
}
```

We use the initWithCoder and encodeWithCoder methods to map our memory game properties to the corresponding keys that will be used to serialize and deserialize our data. With these objects now NSCoding compliant, NSKeyedArchiver and NSKeyedUnarchiver can serialize these objects to/from NSData.

Archiving:

```objc
- (void)saveCurrentGame
{
    if (self.gameInProgress) {
        NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
        NSData *gameData = [NSKeyedArchiver archivedDataWithRootObject:self];
        [defaults setObject:gameData forKey:@"com.memory.game"];
    }
}
```

Unarchiving:

```objc
NSData *savedGameData = [[NSUserDefaults standardUserDefaults] objectForKey:@"com.memory.game"];       
     if (savedGameData) {
         sharedInstance = [NSKeyedUnarchiver unarchiveObjectWithData:savedGameData];
     } else {
         sharedInstance = [[self alloc] init];
     }
```

The flow here is we start with a custom object which is not NSCoding compliant, we add the <NSCoding> tag so the system knows it will adhere to the NSCoding protocol. Then we define the NSCoding methods, initWithCoder and encodeWithCoder. Now that we have an NSCoding compliant object, we can serialize it to/from NSData by using NSKeyedArchiver and NSKeyedUnarchiver. NSKeyedArchiver turns an NSCoding compliant object into NSData which we can then store somewhere like NSUserDefaults. NSKeyedUnarchiver reads an NSData object and turns it back into its original object.

#### NSUserDefaults

You saw above how to create the NSData that will represent our saved game objects. We stored that NSData object in the user defaults object. This is a simple database that is available for each app to store basic settings. It's nice because it provides a quick and easy dictionary-based API to access objects stored within it.

While what we are doing is persisting our custom objects to this database, it's more often used for things like seeing whether an app has launched for the first time, or some flag is true/false etc. I don't recommend persisting your entire object graph to NSUserDefaults but for our use case that's perfectly fine. One other item worth mentioning is to NEVER store sensitive data in here because it's unencrypted and intended only for simple read/write access for publically available settings. Don't put anything in here you wouldn't want exposed to the world.

Read more about NSUserDefaults [here](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSUserDefaults_Class/)

And [here](http://nshipster.com/nscoding/) is a great review of NSCoding and the pros/cons

---

### Commit 14 - 16 - Basic Offline Handling & Other Goodies

Not a whole lot was added in this set of commits. The app can now display a basic error message for the failure block of the networking calls. I also added in an interface to the transition animator so it can handle transitions to the right. It uses exactly the same method as the other one I discussed except it uses a different set of frames. There is also now an about screen. We made this screen using the same techniques as the previous screens but it would be a good exercise to read through and check your understanding.

---

### Commit 17 - Adding another API (Twitter)

This commit adds in the Twitter API so that we can fetch trending topics to search for in Giphy. The goal is to get a list of trends into a table view controller, and have each trend load an example animated GIF from giphy. This trending topic can then be used to populate the animated GIFs in the actual game.

The networking code is all done via a library - STTwitter. It uses the same networking behind the scenes that we used before but makes it easier for us because the Twitter API uses oAuth. You can read about their API [here](https://dev.twitter.com/oauth). Suffice it to say working with oAuth can be burdensome and the STTwitter pod is great. The additional step we need to perform (that the library handles for us) is authenticating our application before our API calls. 

One quick aside: when you are going through these open source libraries and find something that doesn't work - try to fix it. I found that the hashtag exclusion code was not working because Twitter had updated their API. Previously they had wanted a boolean flag value sent, either 0 or 1. Now they've requested it to be a descriptor "hashtags". You can see how simple of a change this is but it keeps the library clean and robust for others to rely on. Be a good open source citizen!

This will get us a list of trends in NYC which we'll use to feed into our tableview:

```objc
- (void)fetchTrends:(void (^)(NSArray *trends))success failure:(void (^)(NSError *))failure
 {
     STTwitterAPI *twitter = [STTwitterAPI twitterAPIAppOnlyWithConsumerKey:TWITTER_CONSUMER_KEY
                                                             consumerSecret:TWITTER_CONSUMER_SECRET
                              ];
     
     [twitter verifyCredentialsWithUserSuccessBlock:^(NSString *username, NSString *userID) {
         
         [twitter getTrendsForWOEID:@"2459115" excludeHashtags:@1 successBlock:^(NSDate *asOf, NSDate *createdAt, NSArray *locations, NSArray *trends) {
             NSMutableArray *allTrends = [[NSMutableArray alloc] init];
             for (NSDictionary *trendDictionary in trends) {
                 NSString *trend = trendDictionary[@"name"];
                 [allTrends addObject:trend];
             }
             success(allTrends);
         } errorBlock:^(NSError *error) {
             failure(error);
         }];
         
     } errorBlock:^(NSError *error) {
         failure(error);
     }];
 }
 ```

The last minor update made is to our Giphy networking interface. It's now split into two methods, one which will fetch the latest trending GIFs (24 of them - for the default game mode of our app) and one which will grab 2 gifs for a preview of any of the trending terms above when needed. We'll need this one when we create our trending topic VC.

```objc
+- (void)fetchGiphyImageDataForSearchTerm:(NSString *)searchTerm success:(void (^)(NSArray *imageData))success failure:(void (^)(NSError *))failure
 {
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

    NSString *url = @"http://api.giphy.com/v1/gifs/search";
    NSMutableDictionary *md = [[NSMutableDictionary alloc] init];
    md[@"q"] = searchTerm;
    md[@"api_key"] = @"dc6zaTOxFJmzC";
    md[@"limit"] = @2;
    
    [manager GET:url parameters:md progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        
        success(responseObject[@"data"]);
        
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        failure(error);
        
    }];
}
```

The difference between this new one and the previous one are that we are now using a parameters dictionary to pass into AFNetworking where previously we gave it the entire URL to fetch. Using a parameters dictionary is just an easier way of constructing the url which would have this tacked on to the end if we made it manually &q=SearchTerm&api_key=dc6zaTOxFJmzC&limit=2. The other nice thing is AFNetworking handles the encoding of our URL for us. We don't have to worry about special characters given back to us by the search term such as white space, punctuation, etc. They will be converted into URL-encoded characters - see [AFURLRequestSerialization](https://github.com/AFNetworking/AFNetworking/blob/master/AFNetworking/AFURLRequestSerialization.m) for more details.

---

### Commit 18 - TableViews & Delegates & Protocols

Ah, our last and final view controller. This will be a fitting end to this somewhat brief series and table views are a perfect topic. If you can understand how a table view works you're most of the way there to becoming a solid junior developer. The aim of a table view is to display a series of data on screen in a vertically scrolling table and do so for either small or large datasets in a memory efficient way. Hey look, I was pretty close to [Apple's description](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableView_Class/)! "A table view displays a list of items in a single column. " 

For our example, we'll be getting a list of trends from Twitter (which we just discussed) and displaying it in a table view. We also have a UIImageView to display the GIFs from Giphy that will correspond to each Twitter trend. I'll leave it up to you to review the non tableview related code as its pretty straightforward.

#### Delegates

Table views make heavy use of the delegate pattern in order to provide great flexibility and customization to the developer. Delegation is a powerful and very common pattern in iOS programming so lets spend some time talking about it. What the delegate pattern aims to do is to seperate a set of concerns or dependencies from one object to another. It lets a parent object delegate out these concerns to the other object (the delegate) and let the delegate worry about the implementation details. The parent object doesn't care how its delegate implements it, only that the delegate implements it. This creates a tight coupling between the two objects but enables great flexibility.

If we were going to setup a table view, lets take a simplistic approach and say we only need to know how many rows of data it should display (it will display the same thing in all of the rows so ignore the content for now). The tableview can't function if it doesn't know how many rows of data to display - so this is a requirement to have before it can work. So how would we do that without delegation? Something like create a table view, and set a property on its numberOfRows to a number?

```objc
UITableView *tableView = [[UITableView alloc] init];
tableView.numberOfRows = [self.dataSource count];
```

Not bad. But what about when our data source changes, maybe we refresh our Twitter API call and now we have a different number of elements in our array. Well that's easy, we'll just set the tableView.numberOfRows again and every time it changes. That's fine, but what if we expand our model table view so that it has to know a bunch of other things in order to function? Like, number of sections, or what each cell should look like? Are we going to setup all of that in advance and update it each time it changes? Something like:

```objc
// called everytime we update some variable that the tableview depends on
- (void)updateTableView
{
    self.tableView.numberOfRows = [self.dataSource count];
    self.tableView.numberOfSections = ...some other thing;
    for (item in self.dataSource) // do something that will create a cell and let the tableview know how to render it

}
```

This is probably feasible but it can quickly spiral out of control. If we take a step back here, with our current approach what we're really doing is pushing information onto the table view every time something changes. Delegation aims to flip this around. It recognizes that the tableview shouldn't be the object that knows about how many rows or sections it has or how the cells should look. That should be the job of some other object. Some object that is much closer to the data. So delegation does this by creating a pull relationship. Our delegate object is going to implement some methods that the parent object can call on to get some of the required information. The table view now has a delegate object that it can ask questions of every time it needs to. The delegate doesn't necessarily know when it will be asked a question, only that it will be asked the question and that it will need to respond appropriately. So with delegation our pattern now looks like this.

```objc
- (void)viewDidLoad 
{
    UITableView *tableView = [[UITableView alloc] init];
    tableView.dataSource = self;
}

// This is a delegate method
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return [self.trends count];
}
```

In this case, self is the UIViewController itself. The UIViewController is now responsible for answering some of the tableviews questions. Remember, a tableview is just a custom view like any other view that we already know how to setup. The difference is that it happens to implement the delegate pattern. With this delegate pattern setup, the tableview object can now ask its delegate, "Hey how many rows should I have?", "Hey, how many sections should I have?", "Hey what should this cell look like?", "What about this one?" etc.

If you're wondering how the tableview delegate code works, it would look something like this extremely simplistically:

```objc
// here is where the object needs to inquire about how many rows it needs to display in section 0
NSInteger theNumberOfRows = [self.delegate tableView:self numberOfRowsInSection:0];
```

It sends a reference of itself to the delegate in case the delegate needs to know what object its working with, and the section number. Now the delegate has to figure out how many rows should be in that section and report back to the table view. Simple, lovely.

So that's it in a nutshell but how does the delegate know what methods to implement?

#### Protocols

A protocol is just a fancy word that defining an interface that contains a set of methods, some required, and some optional. In our case of the table view, it would be a protocol called UITableViewDataSource that defines the required method of numberOfRows, numberOfSections, cellForRowAtIndexPath.. and so on. It creates a contract wherein a delegate knows exactly what sorts of methods it must implement in order to be the delegate of the tableview. You see it like such:

```objc
@interface TrendingViewController () <UITableViewDelegate, UITableViewDataSource>
```

We've implemented the UITableViewDataSource protocol so now we're required to implement the tableview methods or else things will break! By the way, we also implemented the UITableViewDelegate protocol which defines what methods we need to implement so the tableView knows how to display itself and what it should do if a user taps on the cell, etc. Now would be a good time to review the docs on both: 

+ [UITableViewDataSource](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableViewDataSource_Protocol/)
+ [UITableViewDelegate](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableViewDelegate_Protocol/)

#### Back to Table Views

Very quickly, we use the below two methods to tell the tableview the number of sections and rows every time it asks as we just discussed:

```objc
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    return 1;
}

- (NSInteger)tableView:(UITableView *)tableView
 numberOfRowsInSection:(NSInteger)section
{
    return [self.trends count];
}
```

And now that we understand delegation lets get back to our tableview. So we've got the list of trends, and we've setup an array called self.trends in our view controller. This is the first place to start when thinking about table views. What is my data? What do I want to display on screen? Well that's easy, for us its an array of trends. Each trend is just a string which indicates the trend name. Cool - so we know our table view data is going to display an array of NSStrings. We'll hereafter refer to this array as our data source. 

A table view essentially transforms this array of data into, well, a table by taking each slice of data from its data source and displaying it in a row. Each of these rows, which we'll now refer to as cells, corresponds to one index in our array. For each cell, we'll have to setup how we want it to look and what kind of information it should display. This will be straightforward for us as there are sane default cell styles where you get a UITextField that spans the cell (perfect for just displaying one string per cell). Keep in mind that cells are fully customizable and you can roll your own custom cell if you feel like it. So we'll just set the default UITextField text to our string property.

Our tableview will ask its delegate (the TrendingViewController) to give it a cell for each row. The delegate wil figure out what to put in each cell, then return the cell to the tableview so that it can be displayed. Each cell corresponds to one item in the data source. The indexPath is just an object that represents a location in the tableview (generally a section and a row). The dequeueReusableCellWithIdentifier is a pattern that Apple implements in order to be memory friendly. If we have a table with 1,000,000 rows but only 10 are on the screen at a time we shouldn't create 1,000,000 UITableViewCells. We should just create as many cells need to be on screen and once a cell scrolls off screen, we'll reuse it. So we would display 1 - 10, 2 - 11, 3 - 12, 4 - 13, etc. Keep in mind this method is going to be called a lot, in fact each time you see a new cell this method is being called. So be careful of performing expensive operations in here.

```objc
- (UITableViewCell *)tableView:(UITableView *)tableView
         cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"cell" forIndexPath:indexPath];
    
    [self configureCell:cell forRowAtIndexPath:indexPath];
    
    return cell;
}

- (void)configureCell:(UITableViewCell *)cell
    forRowAtIndexPath:(NSIndexPath *)indexPath
{
    cell.selectionStyle = UITableViewCellSelectionStyleNone;
    cell.backgroundColor = [UIColor blackColor];
    cell.textLabel.font = [UIFont fontWithName:@"8BITWONDERNominal" size:18];
    cell.textLabel.textColor = [UIColor whiteColor];
    cell.textLabel.text = self.trends[indexPath.row];
    
    if ([cell.textLabel.text isEqualToString:self.keyword]) {
        cell.textLabel.textColor = [UIColor greenColor];
    }
}

```

So now we have a tableview that does quite a lot: it knows how many rows to display, how many sections, what each row (cell) should look like. Now let's answer the question of what happens when a user selects a cell. Delegation? Yep!

```objc
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    NSString *trend = self.trends[indexPath.row];
    UITableViewCell *cell = [self.trendingTopicsTableView cellForRowAtIndexPath:indexPath];
    cell.textLabel.textColor = [UIColor greenColor];
    
    NSUserDefaults *def = [NSUserDefaults standardUserDefaults];
    
    GiphyNetworkingModel *giphyAPI = [GiphyNetworkingModel sharedInstance];
    [giphyAPI fetchGiphyImagePreviewForSearchTerm:trend success:^(NSURL *url) {
        if (url) {
            self.noGIFsFoundLabel.hidden = YES;
            [self.imageView sd_setImageWithURL:url];
            [def setInteger:0 forKey:@"com.memory.customization"];
            [def setObject:trend forKey:@"com.memory.keyword"];
            self.keyword = trend;
            [def synchronize];
            
        } else {
            self.noGIFsFoundLabel.hidden = NO;
            [self.imageView sd_setImageWithURL:nil];
            [def setInteger:2 forKey:@"com.memory.customization"];
            [def removeObjectForKey:@"com.memory.keyword"];
            [def synchronize];
        }
        
    } failure:^(NSError *error) {
        NSLog(@"Error with giphy");
    }];    
}
```

Every time the tableview intercepts a touch event, it asks its delegate what it should do. It says here is the indexpath of the touch event - go figure out what should happen. So we get the trend represented by the row the user just clicked. We change the textColor to be green so the user gets some feedback and knows where they are spatially. Then we reach out to giphy to get the gif data for that trend. If we get a response, we show it in our UIImageView. If we don't, we throw up the no GIFs found label. We'll also toggle a few of our defaults so that our game knows to use the custom trend if the user has selected one or the default game type if not. 

And that's it! We're done.

---

### Commit 19 - 20 - Final Cleanup

Explore this last set of commits at your own leisure as its nothing new. We just made a few tweaks such as letting users make a new game in case they want to pick a new trend or restart the current game if they get tired of it. A display was also added to the game so the user knows if they're playing a trend or a default game. We used the content compression resistance property for autolayout here to define which views should get priority when autolayout has to make a decision about the width of each of its elements. Check it out, its good stuff.

---

### We're done!

Phew, we've covered quite a lot at a breakneck pace. My goal was simply to expose you to a lot of the concepts and explain them very briefly at a high level. I highly encourage you to read more on each and every one of these topics. They're all very important and I simply cannot do each one of them justice in this short tutorial. Hopefully you can now see how each piece played a role in building this silly little memory game. Play around, see what other fun features you can add to this game or tweak. Or push it to the appstore!
