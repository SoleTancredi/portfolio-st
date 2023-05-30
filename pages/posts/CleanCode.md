Good programmers write boring code. Great programmers write really boring code

When I came across this quote on the internet, sometime in the past year, it introduced me to the concept of clean code.

What is Clean Code exactly?
In simple terms Clean Code refers to a codebase that is easy to read and understand. Some salient features of Clean Code are:

Reduces cognitive load of the reader
Contains intuitively named variables and functions
Follows the best practices for coding
Why use Clean Code when my code works even without it?
Glad you asked. Clean Code is not about writing code that works, it is about writing code that is easy to read and maintain in the long term. As you can see from this graph, the cost of maintaining a Dirty Code base drastically increases over time, whereas in the case of Clean Code, it remains fairly constant.

import Image from 'next/image'

<Image
  src="/images/clean.png"
  alt="Photo"
  width={800}
  height={650}
  priority
  className="next-image"
/>

Clean Code is not the shortest or the smartest looking, but it is the elegant code, understanding which takes minimal effort.

Programming is an art as much as it is a science. Here are a few tips to help you write Clean Code.

## 1. Intuitive Variable Names

Quite evidently nobody does all the computation mentally to check what a variable stores.

## Example 1

```const x = n.filter(e => e > 0);```

## Example 2

```const positiveElements = numbers.filter(num => num > 0);```

As you can see the second example is far easier to understand than the first one.

In the case of booleans the variable name should be a question that can be answered with yes or no, such as

```const isValid = false; \n const hasAuthorization = true;```

2. Self-Explanatory functions
   There are two things to keep in mind while writing functions or methods:

Follow the Single Responsibility Principle (SRP)
Function names should be actions words (verbs)
Single Responsibility Principle (SRP) states that a function should do only one specific task. An example of this would be a function that sends data to the server should not validate the data.

### code 


There are two other good to follow principles:

Function body should not contain more than 2 levels of nesting
A function should take a maximum of 3 arguments


### 3. Group Similar Functions Together
We now come across the concept of Cohesion. Cohesion is a measure of the degree to which the elements of the module are functionally related.

Related functions should be grouped together in a class, helping us keep our code compartmentalized

### CODE

Here we find that the IOHelper class only groups together functions for io operations, without having any utility of its own.

4. Minimize The Number Of Comments
This might sound counterintuitive for some beginners, but hear it out. The code you write should be self-explanatory, anyone viewing your code should not have to rely on comments to understand what it does.

There are a few rare cases where you might need comments in case there is some unintuitive code without a workaround.

### CODE

5. Code Formatting
The codebase should always follow a strict set of formatting rules. It is also a good idea to use a formatter like black or prettier to automate the formatting.

The Project Structure should be decided before the commencement of the project and everyone working on it should be aware of it and agree to abide by it, as the Project Structure is context-dependent and everyone likes a specific structure over another.

The variables and function naming convention should also be specified beforehand (isValid or hasVality, etc.) and language-specific cases should be kept in mind (eg: snake case in python and camel case in JS/TS).

### CODE
### CODE

Conclusion
As mentioned earlier as well, Clean Code is not the shortest code, but it is the easiest to understand. Following these principles would help you become a better coder not by improving your coding skills but by improving the soft skills required for coding as it will teach you how to explain your code to non-technical people.

Research says, writing down your goals on pen & paper makes you 21% to 39% more likely to achieve them. Check out these notebooks and journals to make the journey of achieving your dreams easier:
