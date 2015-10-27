---
published: true
title: Case Study: Blocspot (persisting data with NSCoding)
layout: post
---
Blocpost is the name of an iOS app that built from scratch as one of my Bloc.io apprenticeship portfolio projects. The scope of this particular project boiled down to building an app that leveraged many components of the MapKit framework including the ability for a user to search the MapKit API for specific locations or item categories and have the results of that search (the most relevant per the user’s current location) be presented on a MKMapView with annotations. Beyond this a user is given the ability to save individual annotations or Points of Interest (aka POI’s), assign an individual Category to such POI’s and have those saved POI’s persist across open and closing the application. Other intended functionality included the ability to created new Categories that included an assigned UIColor that would be displayed within the appropriate UI elements associated with the particular category name.

The problem: Where to start?
Being that I was not at all familiar with the MapKit framework thus far in my studies, nearly each feature that the app assignment called for posed some challenges for me. Some research of the overall framework was definitely needed. But in order to get started in the right direction to build such an app (as with any app), I understood that I needed to consider the the overall model or organization for the app. I had to think about things such as: 

What custom classes would be necessary? 
How would instances of those instances relate to one another?  
And how would I persist the data? 

It’s the implementation of that model for the app and my thinking around it that will be the focus of this case study.

the solution:
Fresh off another application project that used CoreData, I immediately thought I would just leverage the same approach for managing persistence within the Blocspot app. However, after reading this recommended NSHiptser article it struck me that CoreData may be little overkill for this new project. And NSCoding might do just fine when considering the few elements users would want to be saving within this app. From the project requirements I understood a user would only be saving elements that could likely managed by two custom classes:

Points of Interest (POI.h)
Categories (Category.h)

So NSCoding it was. But where was I going to save these custom objects? I decided a singleton “DataSource” object was right for the job. And within this class I thought just two NSMutableArray properties would do the trick for storing the user’s saved data (at least to start the app project).

//Example code of  DataSource.h file interface.
{% highlight objective-c %}
{% endhighlight %}

In the above code snippet you can see that I immediately import my two other custom model classes and I’m sure to declare a custom class method that will be used for initializing my singleton “sharedInstance”. I also declare a method called saveData that of course will be needed to save the user data by leveraging NSCoding. 

Within the implementation file, I want to start with defining this saveData method because elements within this method are important for how the app retrieves and unpackages the saved data using NSCoding’s NSKeyedUnarchiver at app launch. Those important elements are the file paths that I create for both the user’s saved Points of Interest and their saved Categories as shown here:

//Example code of saveData method
{% highlight objective-c %}
{% endhighlight %}

Now at the top of my .m file I begin with defining my singleton instance using the recommended style that I learned from my studies (nothing fancy here). 

//Example code of my singleton
{% highlight objective-c %}
{% endhighlight %}

And immediately following this I define the the custom init method which of course outlines how this single instance of the DataSource class will construct itself. In most cases the answer to that will be the saved data (aka the user’s saved Points of Interest and individual Categories) which is achieved by creating a file path and confirming if that specific file path already exists on disk. In the case of Blocspot app, the two file paths we’re interested in end with “poi” or “category” as we defined in our saveData method earlier. But what if the user doesn’t have any saved data? That would certainly occur when the user opens the app for the very first time. In order to accommodate for this scenario and others I implemented several if/else statements to cover:
The file path doesn’t exist (therefore create a new mutable array and set it equal to the app’s property for storing Points of Interest saved by the user).
The file path does exist and the data array found by the NSKeyedUnarchiver contains at least one object (therefore set the user’s saved Points of Interest array equal to that)
The file path does exist but the data array found by the NSKeyedUnarchiver is empty (therefore create a new mutable array and set it equal to the app’s property for storing Points of Interest saved by the user).

And of course we do something similar for unarchiving the user’s Category objects. See below:

//Example code of my DataSource init method:
{% highlight objective-c %}
{% endhighlight %}


You might notice that I’m calling a class method when populating the sharedInstance’s self.categories array property if the unarchiver finds the savedData array for category objects is empty. The reason? I wanted users of the app to at least begin with a couple of category options at app launch and they could build upon this list if they so chose. 

This class method is of course defined in the Category implementation file. In this same file I of course also implement the necessary <NSCoding> protocol methods for saving individual properties and instantiating Category objects with those properties with saved values. 

initiWithCoder & encodeWithCoder

For example, the category name (of type NSString) was an important saved property value that I wanted to persist across app sessions. That was accomplished like so:

{% highlight objective-c %}
- (instancetype) initWithCoder:(NSCoder *)aDecoder {
    self = [super init];
    
    if (self) {
        self.categoryName = [aDecoder decodeObjectForKey:NSStringFromSelector(@selector(name))];
       
    }
    return self;
}


- (void) encodeWithCoder:(NSCoder *)aCoder {
    [aCoder encodeObject:self.categoryName forKey:NSStringFromSelector(@selector(name))];
   
{% endhighlight %}

I applied the same initWithCoder and decodeWithCoder methods to other important properties of the Category class and the POI class as well of course. But in the end the results were the same. A user of the Blocspot app could save individual Points of Interests and Categories of their liking and have those values persist. Thank you NSCoding!

In a future case study of this same app, I’d like to to highlight how I implemented several components of the MapKit framework. 
