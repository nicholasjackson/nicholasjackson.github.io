---
layout: post
title:  "Swift Unit Tests and Access Modifiers"
date:   2014-08-08 16:00:00
categories: swift ios
comments: true
---
### Introduction
XCode 6 beta 4 introduced access modifiers to **Swift**, OK access modifiers were in the previous versions but they technically did nothing more than produce a compiler warning.  For those not familiar they are...

* **Public** access enables entities to be used within any source file from their defining module, and also in a source file from another module that imports the defining module. You typically use public access when specifying the public interface to a framework.
* **Internal** access enables entities to be used within any source file from their defining module, but not in any source file outside of that module. You typically use internal access when defining an app’s or a framework’s internal structure.
* **Private** access restricts the use of an entity to its own defining source file. Use private access to hide the implementation details of a specific piece of functionality.

### Access Modifiers and TDD

So we are writing our code following the TDD methodology, we have a class and that class has a method, the function of which we need to write tests for.  When writing object oriented code you should try to only expose the methods that define your usable interface and this should be kept as small as possible.

We do this because at some point you are going to want to refactor your class and the smaller the impact it has on the rest of your code the simpler it will be to refactor.  What's that I hear you say, it's my code and I know what I'm doing so I'm not going to have that problem.  Take it from me you will, even you will forget the function of some classes and methods as your code grows in complexity and this is where the rot sets in.  

Do your self a favour and follow the below principle:

#### "as public as necessary as private as possible"

Here some simple swift code and a simple test for it...  
{% highlight javascript %}
class MyClass {

    func myFunc() -> Int {
        return 500
    }

}

import UIKit
import XCTest
import Test

class TestTests: XCTestCase {

    override func setUp() {
        super.setUp()
        // Put setup code here. This method is called before the invocation of each test method in the class.
    }

    override func tearDown() {
        // Put teardown code here. This method is called after the invocation of each test method in the class.
        super.tearDown()
    }

    func testExample() {
        // This is an example of a functional test case.
        let clazz = MyClass()
        XCTAssert(500, clazz.myFunc())
    }
}
{% endhighlight %}

In XCode the first class would be defined in MyClass.swift and the test defined in TestTests.swift and as of *beta 4* the default protection level for classes and methods is *Internal*, this used to be *Public* in beta 3 and below.

The default for XCode is that you add your tests classes to the test target and your application classes under test to the application target, the main application then referenced as a target dependency in the XCode build phases panel (see below).
<br/><br/>
![XCode Build Phases]({{ site.url }}/images/post_images/xcode-tdd-swift.png)
<br/><br/>

If you run the tests in XCode with this default setup under beta 4+ the code will not even compile as you will be getting unresolved identifier errors.  This is due to the fact that the test target is different to the application target and thus can not access Internal classes and methods.

#### What are you going to to do about that then?
Make all the methods public, Objective C had no issue with this right?  
I'm not even going to have this argument, we have already ascertained that access modifiers are good and as an engineer you want to write great code not just working code, don't you?  Personally I like access modifiers and I think using them in conjunction with a TDD based approach allows you to naturally create good structured code, so drop this idea and find another solution.  I find that if you don't follow *SOLID* principles when writing code it is really difficult to test therefore we can develop a simple rule here when coding....

#### "If its difficult to test, its probably not the best code"

### The Solution
OK, just tell me how to test bloody internal modifiers.  It's actually quite simple, just add the class under test to the test target's compile sources list.  It's possible this requirement will change in a future XCode release but for now its a fairly painless additional step.
