---
layout: post
title:  "Custom Elements in TVML"
date:   2016-03-23 20:31:20 +0100
categories: swift
---

If you are not familiar with TVML it’s a markup language created for apps running on the new Apple TV which allows you to describe native interfaces similar to HTML without writing a single line of Swift.

TVML documents can also be fetched from a server making them ideal for “browsing interfaces” where the content changes frequently and the interface does not need to be highly customized.

TVML templates provide a great way to kickstart your application development by providing pre-built interfaces for common patterns in an easy to use language.

While this is very useful to get your foot it the door you might  eventually want to advance your application to a point where the stock TVML elements can't help you anymore.

Let’s learn how to extend TVML to your needs by building a few custom components.

## Meet the members of TVMLKit
Apple’s [TVMLKit Reference](https://developer.apple.com/library/prerelease/tvos/documentation/TVMLKit/Reference/TVMLKit_Collection/) does not go too much into the details. It barely even covers how the different classes are supposed to be connected. So let’s do a brief introduction to all the classes your need to get your view rendered.

### [TVViewElement](https://developer.apple.com/library/tvos/documentation/TVMLKit/Reference/TVViewElement_Ref/index.html#//apple_ref/occ/cl/TVViewElement)
This class describes your element inside the DOM. It allows you to access its attributes and styles, dispatch JavaScript events, and access its child and parent element.
Your can regard the TVViewElement as kind of a mediator between your native code and its DOM representation.

### [TVInterfaceFactory](https://developer.apple.com/library/tvos/documentation/TVMLKit/Reference/TVInterfaceFactory_Ref/index.html#//apple_ref/occ/cl/TVInterfaceFactory)
You will subclass this class to build your own Interface Factory. After it has been registered by calling `TVInterfaceFactory.sharedInterfaceFactory().extendedInterfaceCreator` in your AppDelegate it will be called for every element in the current TVML Document.

This allows you to filter out your custom views and then provide a `UIView` or `UIViewController` for them.

### [TVElementFactory](https://developer.apple.com/library/tvos/documentation/TVMLKit/Reference/TVElementFactory_Ref/index.html#//apple_ref/occ/cl/TVElementFactory)
You will access this class in your AppDelegate to register your custom elements by providing their TVViewElement class and their DOM Name (meaning that if you register it as `”myElement”` it will become availible as `<myElement></myElement>` in TVML.


## Set the stage
Let’s connect these parts by building a little sample project which you can download [here](https://github.com/tape-tv/Custom-TVML-Views).

In the project we will create two new elements:

 - `<rainbowText>` will render the text it contains as a rainbow
 - `<rainbowProgress>` will render a progress bar in the colors of a rainbow

### Creating `rainbowText`
The `rainbowText` element will render the characters of its contained text in different colors.

#### Building a Factory
The first thing we need to build is a factory. As stated earlier we will use this class to provide the views for our custom elements.

Start by adding a new class that extends from `TVInterfaceFactory` like so:

{% highlight swift %}
// InterfaceFactory.swift

import TVMLKit

class InterfaceFactory: TVInterfaceFactory {

}
{% endhighlight %}

Since TVInterfaceFactory only accepts a single `extendedInterfaceCreator` we make our class a singleton to avoid confusion.

{% highlight swift %}
// InterfaceFactory.swift

import TVMLKit

class InterfaceFactory: TVInterfaceFactory {
	static var sharedExtendedInterfaceFactory = InterfaceFactory()
}
{% endhighlight %}

We also want to override the `viewForElement` function to be able to return our own views.

This method will be called for every element in the current TVML Document. Since its return value is an Optional we can respond in two different ways:

 - If we want to provide a view for the element, we return a UIView.

 - Otherwise we return nil and the system will take care of it

*Note: Technically, you could also return nil if your element does not display a view to the user (like a background audio element for example)*

{% highlight swift %}
// InterfaceFactory.swift

import TVMLKit

class InterfaceFactory: TVInterfaceFactory {
	static var sharedExtendedInterfaceFactory = InterfaceFactory()

	override func viewForElement(element: TVViewElement, existingView: UIView?) -> UIView? {
		return nil
	}
}
{% endhighlight %}

Now that we have created our factory, let’s register it in the AppDelegate.

In your `application:didFinishLaunchingWithOptions:` method stick this right before the return:

{% highlight swift %}
// AppDelegate.swift

TVInterfaceFactory.sharedInterfaceFactory().extendedInterfaceCreator = InterfaceFactory.sharedExtendedInterfaceFactory
{% endhighlight %}

If you put a breakpoint on the `return nil` line and run the app you will see that the function is being called for every element in the TVML DOM.

#### Registering a custom element
Now let’s register our own element! If your take a look at the [TVMLKit reference](https://developer.apple.com/library/tvos/documentation/TVMLKit/Reference/TVMLKit_Collection/index.html#//apple_ref/doc/uid/TP40016429) again you will see that there are three classes to choose from:

 - `TVViewElement`
	 - `TVImageElement`
	 - `TVTextElement`

Most of the time you will be creating a subclass of `TVViewElement` (will will get to that later).

In this simple case `TVTextElement` provides everything we need (access to the text).

Go back to your AppDelegate and add this line right above the `extendedInterfaceCreator` one:

{% highlight swift %}
// AppDelegate.swift

TVElementFactory.registerViewElementClass(TVTextElement.self, forElementName: "rainbowText")
{% endhighlight %}

This tells TVMLKit to register a new DOM element called “rainbowText” with a `TVTextElement` class. That mean that you can now write

{% highlight swift %}
<rainbowText>Hello World!</rainbowText>
{% endhighlight %}

in your TVML code and it will be recognized!


#### Returning the view
Let’s go back to the `InterfaceFactory` and make it return a view for our new element.

First of all you need to figure out when the view for your element is requested. The implementation of this depends a little bit on how many custom elements you plan to add. For a single element an if-clause might be enough. For more complex elements and hierarchies you might want to build something more flexible. 

For a project with around 3-10 custom elements we found that a switch case is both simple and extendable.

{% highlight swift %}
// InterfaceFactory.swift

override func viewForElement(element: TVViewElement, existingView: UIView?) -> UIView? {
  switch element {
	default:
    return nil
	}
}
{% endhighlight %}

Thanks to Swift’s powerful matching syntax we can filter for our element like so:

{% highlight swift %}
case let element as TVTextElement where element.elementName == "rainbowText":
{% endhighlight %}

That means we bind the constant `element` to a `TVTextElement` that has its `elementName` property set to `"rainbowText"`.

Here is a simple implementation of a rainbow label:

{% highlight swift %}
// InterfaceFactory.swift

override func viewForElement(element: TVViewElement, existingView: UIView?) -> UIView? {
  switch element {
	case let element as TVTextElement where element.elementName == "rainbowText":
		guard let attributedText = element.attributedText else { return nil }
      
      let rainbowString = NSMutableAttributedString(attributedString: attributedText)
      
      for position in 0..<rainbowString.length {
        rainbowString.addAttribute(
          NSForegroundColorAttributeName,
          value: UIColor(
            hue: CGFloat(position) / CGFloat(rainbowString.length),
            saturation: 1.0,
            brightness: 1.0,
            alpha: 1.0
          ),
          range: NSMakeRange(position, 1)
        )
      }
      
      let rainbowLabel = UILabel(frame: CGRect.zero)
      rainbowLabel.attributedText = rainbowString
      
      return rainbowLabel
	default:
    return nil
	}
}
{% endhighlight %}

We won’t go too much into the detail of how this works. If you have questions, write a tweet to [@tape_tv_dev](https://twitter.com/tape_tv_dev)!

`TVTextElement` has an `attributedText` property which returns the node’s text child with the element’s `style` properties applied to it as an `NSAttributedString`. Convinient!

Now if you run your code you should be greeted with a lovely rainbow text! 🌈

### Creating a rainbow progress bar

Next up we will create a more complex example.

Create a new class called `RainbowProgressElement` that extends from `TVViewElement`.

{% highlight swift %}
// RainbowProgressElement.swift

import TVMLKit

class RainbowProgressElement: TVViewElement {

}
{% endhighlight %}

And then register that class for the `”rainbowProgress”` element.

{% highlight swift %}
// AppDelegate.swift

TVElementFactory.registerViewElementClass(RainbowProgressElement.self, forElementName: "rainbowProgress")
{% endhighlight %}

and add it to your TVML code

{% highlight html %}
<rainbowProgress/>
{% endhighlight %}

Next, go back to the `InterfaceFactory` and add a new case statement for the element

{% highlight swift %}
// InterfaceFactory.swift

case let element as RainbowProgressElement:
{% endhighlight %}

This time we only need to match for our class.

This post won’t go into how to create the rainbow progress bar. You can find its code alongside some documentation inside the sample project. Let’s just create the view and return it:

{% highlight swift %}
// InterfaceFactory.swift

case let element as RainbowProgressElement:
	let rainbowProgressView = RainbowProgress(frame: CGRect(x: 0, y: 0, width: 100, height: 10))

	rainbowProgressView.progress = 0.5

	return rainbowProgressView
{% endhighlight %}

#### Implementing style attributes

Of course that still is oddly specific. We want to be able to provide our own width, height, and margins in CSS and provide the progress value from within TVML.

This is where `element` comes into play. As stated earlier the `TVViewElement` acts as a mediator between the DOM element and the native code. That means it allows you to access the values of the DOM element!

You can get the style properties from `element.style`. However, if the values have not been set explicitly they will be replaced by default values (like `0` for the `width` and `height` properties). A way to get around this is to call the `valueForStyleProperty(name: String) -> AnyObject?` function as it will properly return `nil` if the attribute has not been set.

In order to keep a strict separation of concerns, we will add an `explicitWidth` and an `explicitHeight` property to the `RainbowProgressElement` that returns and casts our value.

{% highlight swift %}
//  RainbowProgressElement.swift

var explicitWidth: CGFloat? {
	return style?.valueForStyleProperty("width") as? CGFloat
}
  
var explicitHeight: CGFloat? {
	return style?.valueForStyleProperty("height") as? CGFloat
}
{% endhighlight %}

Now we can create the view’s frame using the values provided from its element. Nice!
 
{% highlight swift %}
// InterfaceFactory.swift

let width = element.explicitWidth ?? 200
let height = element.explicitHeight ?? 10

let rainbowProgressView = RainbowProgress(frame: CGRect(x: 0, y: 0, width: width, height: height))
{% endhighlight %}

*Note: The downside of this solution is that the default value cannot be inspected from within the TVML context (e.g. by using JavaScript or the Web Inspector). So far I have not found an approach to do this as you cannot set the `style` property yourself.*

Let’s try the same thing with the `margin` property. `element.style?.margin` returns a `UIEdgeInsets` struct which we can assign to the views `layoutMargins` property.

{% highlight swift %}
// InterfaceFactory.swift

if let margin = element.style?.margin {
  rainbowProgressView.layoutMargins = margin
}
{% endhighlight %}

Try it out!

{% highlight html %}
<rainbowProgress style="margin: 20;"/>
{% endhighlight %}

#### Implementing custom attributes
The last thing we need is the ability to set the progress value from within TVML. Let’s get started by adding the attribute to the element.

{% highlight html %}
<rainbowProgress progress="0.2" style="margin: 20;"/>
{% endhighlight %}

Now all we need is a way to access that value.
The value is availible in the `attributes` dictionary of the `element`.
To make things a bit cleaner, we can add a value to our `RainbowProgressElement` class that automatically casts the value and returns an optional holding the value or `nil` if the cast failed.

{% highlight swift %}
// RainbowProgressElement.swift

var progress: Float? {
  guard let progress = attributes?["progress"] else { return nil }
  return Float(progress)
}
{% endhighlight %}

Back in our `InterfaceFactory` we set the value

{% highlight swift %}
// InterfaceFactory.swift

rainbowProgressView.progress = element.progress ?? 0.0
{% endhighlight %}

Et voilà! You have created your first real custom component.

TVML can be extremely powerful and this tutorial barely even touches the surface of the topic.
For example we didn’t talk about event handling, custom templates, custom styles, or element hierarchies.
If you are interested in any of these topics, give us a shout at [@tape_tv_dev](https://twitter.com/tape_tv_dev) and we will take it from there.  

