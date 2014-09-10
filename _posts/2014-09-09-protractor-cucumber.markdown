---
layout: post
title:  "Functional testing with cucumber and protractor"
date:   2014-09-09 00:00:00
categories: agile bdd angular protractor cucumber
comments: true
---

Now I love Behaviour Driven Development I love it for many reasons but my main reason is the way it completes my development process.  TDD allows me to test how my code works to validate that the new function has been implemented correctly, to put me in the mindset of writing minimalist and well structured code and to give me confidence I am not breaking any existing code.  It is not so great at describing behaviour that the system is supposed to exhibit and if you work with an Agile methodology it is a pretty terrible interface between the developer and the customer.

### What is BDD?
BDD was developed by Dan North to create some shared tools that both developers and customers could use.  In an Agile world the product owner or customer can sit with the developers and write the features that describe the behaviour they would like the system to exhibit.  Working in this way it helps the developer to get clarity on the problem and it also helps define a group of statements that can later be used as acceptance criteria.

### Gherkin
The most common implementation is to use the Gherkin language, Gherkin is a domain specific language which uses simple grammer to describe behaviour without having to detail how it is implemented.  Below is a simple example that might define some of the functinality for a login page:

{% highlight gherkin %}
Feature: Login Page
  As a user of the site
  I should be able to login

  Scenario: Login Page Title
    Given I go on "login.html"
    Then the title should equal "Login"

  Scenario: Login With In-Correct Details
    Given I go on "login.html"
    When I enter incorrect user credentials
    Then I should see the message "Either your username or password are incorrect!"
{% endhighlight %}

There is no complex code going on here, the feature file reads in simple plain English and the developer has sat with the customer to write this so there is no area for misinterpretation.  This on its own is pretty useful as it's a nice uniform way to define acceptance criteria.  The real beauty of Gherkin however is when you use it with an automated test suite, this allows the developer to write code using an outside in methodology, repeatedly and predictably testing the implementation.

### Outside in development
![XCode Build Phases]({{ site.url }}/images/post_images/outside-in-development.png)

The developer starts with the failing feature and then working scenario by scenario incrementally goes about getting the steps to pass.  In order to get the steps to pass they will most likely have to write some failing unit tests and thus some code to satisfy those unit tests.  Once all scenarios in the feature are passing then the acceptance criteria for the story has been met and the story can be considered done.  The bottom line on this process is quality code, but you don't care about quality right because you love repeating yourself 1000s of times going over the same bug.

### Automated test runners
There are a million and one different test runners that implement Gherkin (Cucumber, RSpec, the original JBehave) therefore it can be used to test everything from raw code, APIs, Websites and Mobile Applications.  Personally I use this approach for all of the afore-mentioned applications and we have even written a specific test runner to test the Starling framework mobile applications.  One of the more common test runners you will find used is Cucumber and other than to bore you about my love for BDD I want to show how Cucumber can be used to test your AngularJS based web applications.

### Enter AngularJS
AngularJS is a Javascript framework for the modern web, it allows you to build dynamic views using a unified and tested framework.   You can data bind your HTML to Restful services using an MVVM model and a whole host of other things which I am not even going to pretend I understand.  One of the great features about Angular is that it has been written from a testability perspective, live services can be easily replaced with test data and it supports dependency injection out of the box so it really helps with writing clean well structured bug free code.

### AngularJS testing with Protractor
So Angular has our unit testing covered but how are we going to test our behaviour, I guess we will just use cucumber and selenium webdriver?  Well you could but it will cause you pain as Angular is an asynchronous framework, just because the page has loaded does not mean that Angular has completed the layout.  This would normally mean you are going to have to handle the load completion yourself, writing lots of ugly unpredictable sleep statements into your tests.  The kind people responsible for Angular have thought about this and created the protractor framework.  Protractor understands Angular directives and its async behaviour so you don't have to write any extra code.  It gives you the ability to access your repeaters and directives directly so no more messy CSS or DOM selectors.

If you want to check how many items you have on a page it becomes as simple as the below example:
{% highlight javascript %}
describe('angularjs homepage todo list', function() {
  it('should add a todo', function() {
    browser.get('http://www.angularjs.org');

    var todoList = element.all(by.repeater('todo in todos'));
    expect(todoList.count()).toEqual(3);
    expect(todoList.get(2).getText()).toEqual('write a protractor test');
  });
});
{% endhighlight %}

### Protractor with Cucumber
The above example uses the Jasmine framework, whilst this is fine for developers it's not the greatest business interface.  Worry not, out of the box Protractor now supports Cucumber, this means you can write your Gherkin files and Cucumber will deal with the interface of these to your javascript specs.  What this gives you is the ability to easily create a framework for testing your application using regular expressions which maximises re-use and retains the all important connection to the customer.

Consider the below feature file:
{% highlight gherkin %}
Feature: Running Cucumber with Protractor
  As a user of Protractor
  I should be able to use Cucumber
  to run my E2E tests

  Scenario: Wrapping WebDriver
    Given I go on "index.html"
    Then the title should equal "My AngularJS App"
{% endhighlight %}
This translates directly into the javascript implementation file listed below:
{% highlight javascript %}
//http://chaijs.com/
var chai = require('chai');

//https://github.com/domenic/chai-as-promised/
var chaiAsPromised = require('chai-as-promised');
chai.use(chaiAsPromised);

var expect = chai.expect;

module.exports = function() {
  this.Given(/^I go on "([^"]*)"$/, function (arg1, callback) {
    // Write code here that turns the phrase above into concrete actions
    browser.get('http://localhost:9001');
    callback()
  });

  this.Then(/^the title should equal "([^"]*)"$/, function (arg1, callback) {
    // Write code here that turns the phrase above into concrete actions
    expect(browser.getTitle()).to.eventually.equal(arg1).and.notify(callback);
  });
}
{% endhighlight %}
As you can see where I have used words like "index.html" in my feature file this has been replaced with a regular expression in my Javascript.  This means I can re-use this step file anywhere I need to write "Given I go on 'page.html'", pretty flexible and pretty awesome.  Ok these steps are not so useful for the business but they are not meant to be this is for you; the feature file is your contract to the business.

### Give it a go I dare you
Protractor and Cucumber are not without their faults but this is a million steps forward from where we were a year ago.  If you are a developer writing web apps with the Angular framework then you are most likely already using Protractor.  If you are not using Cucumber give it a go and see how it improves your relationship with your customer.  If you are the customer and you are not already using BDD, I can't recommend it enough. I guarantee you will see positive results with your software quality and a better relationship with your developers.  Like coding, testing is a skill, it's not easy and takes a little time to learn and implement. Once you have mastered the basics you will never look back.

For your convenience I have created a simple example which uses a scaffold Angular app, you can get it right here, right now :)

### Source Code:
[https://github.com/nicholasjackson/protractor-cucumber-example](https://github.com/nicholasjackson/protractor-cucumber-example)
