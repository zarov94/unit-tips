# Writing testable code

## Table of contents

## Keep Business Logic and Display Logic Separate

One of the primary jobs of a JavaScript-based browser application is listening to DOM events triggered by the end user, and then responding to them by running some business logic and displaying the results on the page. It’s tempting to write an anonymous function that does the bulk of the work right where you’re setting up your DOM event listeners. The problem this creates is that you now have to simulate DOM events to test your anonymous function. This can create overhead both in lines of code and the time it takes for tests to run.

Instead, write a named function and pass it to the event handler. That way you can write tests for named functions directly and without jumping through hoops to trigger a fake DOM event.

This applies to more than the DOM though. Many APIs, both in the browser and in Node, are designed around firing and listening to events or waiting for other types of asynchronous work to complete. A rule of thumb is that if you are writing a lot of anonymous callback functions, your code may not be easy to test.


```javascript
// hard to test
$('button').on('click', () => {
    $.getJSON('/path/to/data')
        .then(data => {
            $('#my-list').html('results: ' + data.join(', '));
        });
});

// testable; we can directly run fetchThings to see if it
// makes an AJAX request without having to trigger DOM
// events, and we can run showThings directly to see that it
// displays data in the DOM without doing an AJAX request
$('button').on('click', () => fetchThings(showThings));

function fetchThings(callback) {
    $.getJSON('/path/to/data').then(callback);
}

function showThings(data) {
    $('#my-list').html('results: ' + data.join(', '));
}
```

## Construct and init

* A constructor should initialize an object in a way that it's in a usable state.

* A constructor should only initialize an object, not perform heavy work.

* A constructor should not directly or indirectly call virtual members or external code.

So in most cases an Initialize method shouldn't be required.

In cases where initialization involves more than putting the object into a usable state (e.g., when heavy work needs to be performed or virtual members or external need to be called), then an Initialize method is a good idea.


## Singleton and Dependency injection

To write testable code, we must separate object creation from the business logic and singletons, by their nature, prevent this by providing a global and a static way of creating and obtaining an instance of their classes. They make it impossible to mock and isolate methods that depend on them for testing. The new operator is no different and same arguments apply. Dependency injection frameworks centralize object creation and separates it from the business logic making it possible to isolate methods and write good unit tests. Projects of all sizes can benefit from DI frameworks. Normally include a DI framework form the very start because it’s tough to resist the singleton temptation half-way through the project when you really need it and don’t have time to mess around setting up DI.

Singleton
```javascript
something: function (parameter) {
    Object.getInstance().doSomething(parameter);
}
```

Dependency Injection
```javascript
something: function (parameter, object) {
    object.doSomething(parameter);
}
```

One of the main benefits of using dependency injection is that you can pass in mock objects from your unit tests that don’t cause real side effects (in this case, updating database rows) and you can just assert that your mock object was acted on in the expected way.

## Give Each Function a Single Purpose

Break long functions that do several things into a collection of short, single-purpose functions. This makes it far easier to test that each function does its part correctly, rather than hoping that a large one is doing everything correctly before returning a value.

In functional programming, the act of stringing several single-purpose functions together is called composition. Underscore.js even has a function _.compose, that takes a list of functions and chains them together, taking the return value of each step and passing it to the next function in line.

```javascript
// hard to test
function createGreeting(name, location, age) {
    let greeting;
    if (location === 'Mexico') {
        greeting = '!Hola';
    } else {
        greeting = 'Hello';
    }

    greeting += ' ' + name.toUpperCase() + '! ';

    greeting += 'You are ' + age + ' years old.';

    return greeting;
}

// easy to test
function getBeginning(location) {
    if (location === 'Mexico') {
        return '¡Hola';
    } else {
        return 'Hello';
    }
}

function getMiddle(name) {
    return ' ' + name.toUpperCase() + '! ';
}

function getEnd(age) {
    return 'You are ' + age + ' years old.';
}

function createGreeting(name, location, age) {
    return getBeginning(location) + getMiddle(name) + getEnd(age);
}
```


## General stuff to watch in review

#### Flaw #1: Constructor does Real Work

Warning Signs

* new keyword in a constructor or at field declaration
* Static method calls in a constructor or at field declaration
* Anything more than field assignment in constructors
* Object not fully initialized after the constructor finishes (watch out for initialize methods)
* Control flow (conditional or looping logic) in a constructor
* Code does complex object graph construction inside a constructor rather than using a factory or builder
* Adding or using an initialization block

#### Flaw #2: Digging into Collaborators

Warning Signs

* Objects are passed in but never used directly (only used to get access to other objects)
* Law of Demeter violation: method call chain walks an object graph with more than one dot(.)
* Suspicious names: context, environment, principal, container, or manager

#### Flaw #3: Brittle Global State & Singletons

Warning Signs

* Adding or using singletons
* Adding or using static fields or static methods
* Adding or using static initialization blocks
* Adding or using registries
* Adding or using service locators

#### Flaw #4: Class Does Too Much

Warning Signs

* Summing up what the class does includes the word “and”
* Class would be challenging for new team members to read and quickly “get it”
* Class has fields that are only used in some methods
* Class has static methods that only operate on parameters