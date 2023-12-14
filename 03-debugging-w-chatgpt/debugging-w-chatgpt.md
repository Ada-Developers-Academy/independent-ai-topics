# Debugging with ChatGPT

## Goals

We have a number of tools in our belt for debugging and understanding code, but there’s always room for more! This lesson is about what we can do with LLMs and generative AI tools like ChatGPT if we’ve used all other debugging tools at our disposal and are still feeling stuck or unsure of what a piece of code is doing.

Our goals for this lesson are to:
- Learn how to effectively share code and context with ChatGPT
- Use ChatGPT in conjunction with our other debugging skills 
- Examine how to evaluate ChatGPT provided explanations for correctness

## Caveats & Concerns

First and foremost, we want to note that all of our debugging skills that we learned before are still necessary and helpful, even with generative AI tools. In many instances, we’ll be able to find and fix a bug faster by reading through the code or adding a couple break points than it would take to craft and iterate on a prompt to get an explanation. Furthermore, because tools like ChatGPT can fabricate information and are not running code themselves to ensure what they suggest works, we need to keep up our debugging and reasoning skills to evaluate what ChatGPT shares with us to see if it 1) makes sense and 2) truly addresses the initial problem.

Second, and very importantly: depending on what it is you’re working on, you may not be allowed to send the code to ChatGPT or other generative AI tools. Enterprise solutions are being created and improved that allow companies to feed their proprietary code into an LLM-based tool without the code being used for further training or made in some way available to other people, but this is not the case for most publicly available LLMs and generative AI tools at this time. 

Be very careful about what you share with LLMs - an example of code that you could submit to ChatGPT to receive an explanation of concepts or debug with would be a personal project that you have complete control over. Any project which you don’t have full ownership over should be discussed and approved by other stakeholders before code is shared with generative AI tools of any kind.

## Tips for working with code in LLMs

### Setting the Stage

It’s helpful to set the stage first when working with LLM tools like ChatGPT while coding. We can do this by prompting ChatGPT to "act as an expert in a particular language, helping you to debug". If we know exactly what problem we need help with we can include more details in the prompt, but a starting prompt could look like:

> You are a python expert who will help me debug python code.

### Gathering Information

Adding in ChatGPT doesn’t change our generic debugging steps, but it means that we may want to take some extra notes in a text editor for crafting a prompt if necessary. We should try our best to provide as many details as possible so that ChatGPT does not need to fill in gaps or make assumptions which might result in an incorrect response. 

As we investigate what is happening, we should take notes of the exact issues our code is experiencing such as: 
- compiler errors that the IDE generates 
- runtime exceptions that occur when the code is executing
- any other unexpected behavior in the code that could stem from a logical error.

When we ask ChatGPT for assistance, we need to be descriptive to get useful help. Our prompts for analyzing or explaining code should include:
- a detailed description of what the code is meant to do
- a detailed description of the input, expected result, and actual result
- any other information needed to reliably reproduce the issue being debugged

The type of information we’re gathering likely sounds familiar - when we ask ChatGPT for help we need to collect the same kinds of detailed information and explanation that we would use if we were reaching out to other people for help who may be unfamiliar with the project. 

## Providing Code & Enumerating Lines

### Sharing the “right” code

Many tools like ChatGPT 3.5 don’t have access to outside sources like GitHub to view code, so we need to be prepared to paste the relevant code into our prompt. We need to provide ChatGPT with all the context it requires to understand where the bug is happening, including details like global variables and imports. 

Let’s take a look at an example, say we have the following files in a project for an `Item` and a `Clothing` class.

```
import uuid

class Item:
    def __init__(self, condition=0, id=None):
        self.condition = condition
        if id is None:
            self.id = uuid.uuid4().int
        else:
            self.id = id

    def get_category(self):
        return self.__class__.__name__

    def __str__(self):
        return f"An object of type {self.get_category()} with id {self.id}."

    def condition_description(self):
        if self.condition > 3:
            return "It's great!"
        else:
            return "It stinks!"
```

```
from swap_meet.item import Item

class Clothing(Item):
    def __init__(self, condition=None, id=None, fabric="Unknown"):
        super().__init__(condition=condition, id=id)
        self.fabric = fabric

    def __str__(self):
        return f"{super()._str_()} It is made from {self.condition} fabric."
```

We would like assistance from ChatGPT to debug why the `Clothing` class’s `__str__` function isn’t working. We could paste just the `Clothing` class’s  `__str__` function into our prompt, but that function uses class variables and has a reliance on the imported class `Item` which tools like ChatGPT would have no way of knowing about. 

In this case, by only including the function, we severely limit the tool’s ability to understand the full picture. If we want to debug what is happening in the `Clothing` class’s `__str__` method, then we should include the `Item` class definition, as well as the entire `Clothing` class definition in our prompt. 

### Enumerating Lines

To make it easy for ourselves and ChatGPT to refer to specific lines of the code, we can prompt ChatGPT to enumerate the lines of the code we just gave it. This way, we can refer to those lines of code by number in later prompts.

> Please enumerate the lines of the code I just gave you.

<details>
   <summary>Try out the prompt above with the `Item` and `Clothing` classes, then expand this section to see the prompts we used to share the code and enumerate it along with ChatGPT's responses.</summary> 

Our Prompt:
```
Given this Python code:
'''
import uuid

class Item:
    def __init__(self, condition=0, id=None):
        self.condition = condition
        if id is None:
            self.id = uuid.uuid4().int
        else:
            self.id = id

    def get_category(self):
        return self.__class__.__name__

    def __str__(self):
        return f"An object of type {self.get_category()} with id {self.id}."

    def condition_description(self):
        if self.condition > 3:
            return "It's great!"
        else:
            return "It stinks!"
'''
'''
from swap_meet.item import Item

class Clothing(Item):
    def __init__(self, condition=None, id=None, fabric="Unknown"):
        super().__init__(condition=condition, id=id)
        self.fabric = fabric

    def __str__(self):
        return f"{super()._str_()} It is made from {self.condition} fabric."
'''
Respond with YES if you understand the code.
```

Chat GPT's response:
> Yes, I understand the code. It consists of two classes: `Item` and `Clothing`. The `Item` class has methods to initialize an object, get its category, provide a string representation, and describe its condition. The `Clothing` class is a subclass of `Item` and adds an attribute for fabric in its initialization, along with a customized string representation.
> 
> If you have any specific questions or if there's anything else you'd like assistance with regarding this code, feel free to let me know!

Our Prompt:
```
Please enumerate the lines of the code I shared
```

Chat GPT's response:
> Certainly! Here are the enumerated lines of the code you shared:
>
> **Code Block 1: Item Class**
> ```python
>  1. import uuid
>  2. 
>  3. class Item:
>  4.     def __init__(self, condition=0, id=None):
>  5.         self.condition = condition
>  6.         if id is None:
>  7.             self.id = uuid.uuid4().int
>  8.         else:
>  9.             self.id = id
> 10. 
> 11.     def get_category(self):
> 12.         return self.__class__.__name__
> 13. 
> 14.     def __str__(self):
> 15.         return f"An object of type {self.get_category()} with id {self.id}."
> 16. 
> 17.     def condition_description(self):
> 18.         if self.condition > 3:
> 19.             return "It's great!"
> 20.         else:
> 21.             return "It stinks!"
> ```
> 
> **Code Block 2: Clothing Class**
> ```python
> 23. from swap_meet.item import Item
> 24. 
> 25. class Clothing(Item):
> 26.     def __init__(self, condition=None, id=None, fabric="Unknown"):
> 27.         super().__init__(condition=condition, id=id)
> 28.         self.fabric = fabric
> 29. 
> 30.     def __str__(self):
> 31.         return f"{super()._str_()} It is made from {self.condition} fabric."
> ```
>
> I've included line numbers for reference. If you have any questions or need further clarification, feel free to ask!

</details>  

## Adding ChatGPT to the Debugging Toolbox

The key to effectively using ChatGPT for debugging is that we’re adding a new tool to our toolbox, not replacing the ones we’ve already built. The general steps of our debugging journey should look pretty much the same: we want to determine what is happening, why it’s happening, and how we fix it. 

Let’s take a closer look at the debugging journey to see where we can make the most use of tools like ChatGPT.

### What’s Happening

#### Investigation

When we’re debugging the problem may not always be obvious, so we may have to do some searches about errors, set breakpoints, or add print statements to better understand what kinds of inputs are giving us an issue and why. If our initial searches don’t come up with clear answers, we can ask ChatGPT questions about the error messages or the behavior we are seeing to get a better idea of what is causing the problem.

### Why is it Happening?

#### Understand the Code

We need a solid understanding of how a piece of code works to debug effectively. As we identified what was happening we may have gotten an understanding of the code, but some projects are complex or need a bit more time to study. If we want a clearer picture of the overall code, we can set a timebox for reading through the code line by line, rubber ducking, charting execution, or any other of our prior tools you find helpful. If we hit our timebox, ChatGPT may be able to help us understand the code by answering questions about its purpose, structure, and syntax.

#### Isolate & Reproduce the Problem

Next we want to dig into why the bug is happening. We want to isolate the problem by finding the line or lines of code causing our issue and reliably reproduce the bug so we know how to test for it and confirm the bug was fixed. Our prior tools are a great place to start, stepping through our code or charting the execution to see when unexpected values begin to appear. 

A helpful strategy can be to set a timebox for stepping through the code with a debugger and when that ends, then reach out to ChatGPT for help determining which lines of code are causing the issue. ChatGPT can also help us use other debugging tools more effectively if we ask for guidance. For example, if we pinpoint where the bug is happening after having trouble doing so with a debugger, we can ask ChatGPT “How could we catch the issue with a debugger?”.

Similarly, if we’re having trouble reproducing the bug consistently, we can set a timebox for exploration and research. If the timebox ends and we still don't have our answer, we can see if ChatGPT can help guide us on how to reproduce the issue and suggest test cases to let us reliably test the issue.

### How we Fix it

#### Fix and Test

Once we’ve found and fixed the bug, we need to test our code to make sure the problem has been resolved. If we didn’t already have test cases for this issue, we can write up the edge and nominal cases that we think of, then ask ChatGPT if there are other helpful cases we’ve forgotten. If we’re having trouble understanding how to write the tests for a particular section of code, we can also timebox our research and ask ChatGPT for guidance on testing the code effectively.

## Evaluating ChatGPT’s Responses

As stated before, we can’t wholeheartedly trust everything ChatGPT says. Evaluating a code response from ChatGPT should look pretty similar to evaluating any other information created by a generative AI - we need to look for verifiable pieces of information in the response and use other trustworthy sources to confirm if those pieces of information are true. Check out the lesson “Prompting Basics & Using ChatGPT - Verifying Information from ChatGPT” if you’d like to refresh yourself on strategies for verifying information.

Let’s say that a student, Althea, is working on [the Adagrams project](https://github.com/AdaGold/adagrams-py) and is debugging an `IndexError` that sometimes appears in their `draw_letters` function:

```
def draw_letters():
    distribution = {'A': 9, 'B': 2, 'O': 8, 'C': 2, 'P': 2, 
                    'D': 4, 'Q': 1, 'E': 12, 'R': 6, 'F': 2, 
                    'S': 4, 'G': 3, 'T': 6, 'H': 2, 'U': 4, 
                    'I': 9, 'V': 2, 'J': 1, 'W': 2, 'K': 1, 
                    'X': 1, 'L': 4, 'Y': 2, 'M': 2, 'Z': 1}
    pool = []
    for letter in distribution:
        pool.extend([letter for _ in range(distribution[letter])])

    hand = []
    for _ in range(10):
        index = random.randint(0, len(pool))
        letter = pool[index]
        hand.append(letter)
        pool.remove(letter)

    return hand
```

Althea has already taken steps to add print statements and step through their code with a debugger, and they are pretty sure the issue is somewhere in the for-loop. 

Let’s take a look at Althea’s conversation with ChatGPT and see how they used it for guidance while confirming the facts of the response:

> **Althea**:  
> You are a python expert who will help me debug python code. 

> **ChatGPT**:  
> Of course! I'd be happy to help you debug your Python code. Please provide the code that you're having trouble with, along with a description of the issue you're facing. If the code is long, you can share the relevant parts or the specific error messages you're encountering.

> **Althea**:  
> Given this Python code:
>
> '''  
> ```   
> def draw_letters():  
>     distribution = {'A': 9, 'B': 2, 'O': 8, 'C': 2, 'P': 2,   
>                     'D': 4, 'Q': 1, 'E': 12, 'R': 6, 'F': 2,   
>                     'S': 4, 'G': 3, 'T': 6, 'H': 2, 'U': 4,   
>                     'I': 9, 'V': 2, 'J': 1, 'W': 2, 'K': 1,   
>                     'X': 1, 'L': 4, 'Y': 2, 'M': 2, 'Z': 1}  
>     pool = []  
>     for letter in distribution:  
>         pool.extend([letter for _ in range(distribution[letter])])
> 
>     hand = []  
>     for _ in range(10):  
>         index = random.randint(0, len(pool))  
>         letter = pool[index]  
>         hand.append(letter)  
>         pool.remove(letter)  
> 
>     return hand  
> ```
> '''    
> 
> Respond with YES if you understand the code.

> **ChatGPT**:  
> Yes, I understand the provided Python code. It defines a function called `draw_letters` that simulates drawing 10 random letters from a pool of letters based on their distribution probabilities. The distribution of letters and their respective frequencies is specified in the `distribution` dictionary. The function uses the `random` module to randomly select letters from the pool and adds them to the hand, ensuring that the distribution is maintained.
> 
> If you have any specific questions or if there's something you'd like me to help you with regarding this code, please let me know!

> **Althea**:  
> Please enumerate the lines of code in the `draw_letters` function.

> **ChatGPT**:  
> Certainly! Here are the enumerated lines for the code within the `draw_letters` function:
> 
> 1. ```python
>    distribution = {'A': 9, 'B': 2, 'O': 8, 'C': 2, 'P': 2, 
>                    'D': 4, 'Q': 1, 'E': 12, 'R': 6, 'F': 2, 
>                    'S': 4, 'G': 3, 'T': 6, 'H': 2, 'U': 4, 
>                    'I': 9, 'V': 2, 'J': 1, 'W': 2, 'K': 1, 
>                    'X': 1, 'L': 4, 'Y': 2, 'M': 2, 'Z': 1}
>    ```
> 2. ```python
>    pool = []
>    ```
> 3. ```python
>    for letter in distribution:
>    ```
> 4. ```python
>        pool.extend([letter for _ in range(distribution[letter])])
>    ```
> 5. ```python
>    hand = []
>    ```
> 6. ```python
>    for _ in range(10):
>    ```
> 7. ```python
>        index = random.randint(0, len(pool))
>    ```
> 8. ```python
>        letter = pool[index]
>    ```
> 9. ```python
>        hand.append(letter)
>    ```
> 10. ```python
>         pool.remove(letter)
>     ```
> 11. ```python
>     return hand
>     ```

> **Althea**:  
> The function should always successfully return a hand of ten letters. In reality, I am seeing a bug where I am receiving an "IndexError" sometimes, but not every time. I think the bug is in the loop from lines 6-10, what are possible causes?

> **ChatGPT**:  
> The bug in your code is likely caused by the fact that `random.randint(0, len(pool))` can return the value `len(pool)`, which is one position beyond the last index in the list. Since Python uses zero-based indexing, this leads to an "index out of bounds" error when trying to access `pool[index]`.
> 
> To fix this issue, you should modify the range of the random index to exclude the upper bound, ensuring that it always generates a valid index. Update the line 7 in your code as follows:
> 
> ```python
> index = random.randint(0, len(pool) - 1)
> ```
> 
> This way, `random.randint` will generate an index from `0` to `len(pool) - 1`, inclusive, ensuring that you don't go beyond the bounds of the `pool` list.

This is a great place to pause and ask further questions if we don’t understand any parts of what ChatGPT is suggesting. If we don’t have clarifications, we can move onto evaluating what ChatGPT shared with us.

ChatGPT told Althea that their usage of `random.randint` is likely the culprit - great, we have an answer! But wait, how do we know that changing how they use the function will fix the bug? Before making any changes to their code, Althea goes to the documentation for [Python’s `random` module](https://docs.python.org/3/library/random.html) and takes a look at [the `randint` function description](https://docs.python.org/3/library/random.html#random.randint):

```
random.randint(a, b)  
    Return a random integer N such that a <= N <= b. Alias for randrange(a, b+1).  
```  

Althea might choose to look for other sources since there isn’t a lot of information here, but they take note of the statement that the random integer returned `N` will be less than or equal to the second parameter `b` (put another way, `randint`’s parameter `b` is inclusive rather than exclusive like Python’s `range` function). 

This confirms the explanation that ChatGPT gave - since Althea’s second parameter was `len(pool)` it meant that an index could be chosen that was one past the end of the list, causing the `IndexError` they were sometimes seeing. Althea can now update their code with confidence that what ChatGPT suggested is likely the issue, and run their test suite to confirm that the issue is fixed and hasn’t caused side effects.

After completing the project, Althea has some time and wants to refactor their code. Just like with debugging help, if we ask for help refactoring, we want to use the same critical lens to check if ChatGPT’s response is truthful and solves what we are asking for. Althea came back to their chat above and asks:

> **Althea**:  
> Do you have suggestions for how I can simplify the code for choosing ten random elements from the list `pool`?

> *ChatGPT*:  
> Certainly! You can simplify the code for choosing ten random elements from the `pool` list using the `random.sample` function. The `random.sample` function is designed for this purpose and ensures that you get unique elements from the population.
>
> Here's the updated code:
> 
> ```python
> import random
> 
> def draw_letters():
>     distribution = {'A': 9, 'B': 2, 'O': 8, 'C': 2, 'P': 2, 
>                     'D': 4, 'Q': 1, 'E': 12, 'R': 6, 'F': 2, 
>                     'S': 4, 'G': 3, 'T': 6, 'H': 2, 'U': 4, 
>                     'I': 9, 'V': 2, 'J': 1, 'W': 2, 'K': 1, 
>                     'X': 1, 'L': 4, 'Y': 2, 'M': 2, 'Z': 1}
>     
>     pool = []
>     for letter in distribution:
>         pool.extend([letter] * distribution[letter])
> 
>     hand = random.sample(pool, 10)
> 
>     return hand
> ```
> 
> This code uses `random.sample(pool, 10)` to directly obtain a list of 10 unique elements from the `pool` list, eliminating the need for the loop and manual removal of elements.

We can see that ChatGPT is suggesting that Althea uses a separate function `sample` from the `random` module. Before copy/pasting the suggestion into their project, they go back to the random module documentation or other trusted resources to ensure: 
- Does the `random.sample` function exist? 
- If `random.sample` does exist, does it actually do what ChatGPT says it does? 
- If it does what ChatGPT thinks, then does using the function meet the original code requirements?

In this case, the answer to all of these questions is Yes! Althea can once again update their code and run the test suite to ensure everything is still in working order after their changes 🎉.

## Summary

ChatGPT can be a really helpful tool to give us more insight on what code is doing, but we need to be careful about over-reliance. There are many situations in life such as interviews and code review meetings where we may not have access to generative AI tools, meaning that tools like ChatGPT are best used to support and augment the debugging skills we’ve built rather than to replace them. When we start out a debugging journey, we still want to begin with searching errors on the web, using print statements, setting breakpoints to step through our code, and asking others for help before turning to ChatGPT. 

If we have trouble finding resources about the error or understanding what the code is doing using our prior skills, we can begin to augment our investigation by folding in ChatGPT to see if it can explain an error and common causes, or talk through what particular lines of code are doing in reality vs expectation. 

Before applying any suggestions from generative AI tools we need to break out our information validation skills to check if what they are suggesting is accurate for the language we’re working in and addresses the intended issue. And just like any situation where we’re bug fixing or refactoring, we should be running our tests frequently and expanding our test suite if necessary to ensure that we are truly addressing the issue without causing side effects.

The more we work with code, the more familiar we will be with common edge cases and coming up with the edge cases unique to our project, but we can also ask ChatGPT to help us brainstorm test cases to help strengthen our test suites. 

## Check for Understanding

<!-- Question 1 -->
<!-- prettier-ignore-start -->
### !challenge
* type: ordering
* id: f22a6178-953d-11ee-b9d1-0242ac120002
* title: Ordering Althea's debugging steps
##### !question

Althea is debugging another project that is producing a `TypeError` and they are trying to pinpoint the line causing them trouble. 
Order the debugging steps that they took from first to last action:

##### !end-question
##### !answer

1. Do a web search for the error
1. Go line by line and rubber duck
1. Use a debugger to step through the code
1. Prompt ChatGPT for assistance pinpointing the bug
1. Step through the code to confirm if the line ChatGPT suggested is where unexpected behavior starts to appear

##### !end-answer
##### !explanation

Althea wants to first do some quick searches for the error to see if there are resources on what the error means and why it might be happening. They want to understand the code better, so they do a close reading of the code and do some rubber ducking. Once they feel more clear on the overall goal of the code, they set some breakpoints and step through the code line by line to try to suss out where the strange behavior starts. Althea has an idea of what is going wrong and has narrowed down the issue to a few lines of code but they want more clarity, so they write up what they learned from their observations and prompt ChatGPT for guidance on where the issue might be happening. After carefully reading ChatGPT’s response to ensure it makes sense, they step through their code again, paying close attention to the line that ChatGPT called out as a potential issue and double check that they see the the bug happening at that line.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- Question 2 -->
<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: e3d729c2-9541-11ee-b9d1-0242ac120002
* title: Sharing code with ChatGPT

##### !question

Take a look at [the snowbug repo](https://github.com/AdaGold/snowbug), particularly the source files [main.py](https://github.com/AdaGold/snowbug/blob/main/main.py) and [snowman/snowman.py](https://github.com/AdaGold/snowbug/blob/main/snowman/snowman.py). 

For this problem, let’s say that we’ve fixed all the bugs in `snowman.py` except for one! The losing message always prints at the correct time, but there is something wrong with the string that’s printed. We stepped through the code and are sure there is an issue with the losing message on line 43 of the `snowman` function in the file `snowman/snowman.py`. 

```
print(f"Sorry, you lose! The word was {snowman}")
```

If we wanted to craft a prompt to help debug this specific issue, what is the *least* amount of code we could share with a LLM tool that would give it enough context to help us debug? Choose one option below:

##### !end-question

##### !options
* Line 43 from `snowman/snowman.py`
* The `snowman` function in `snowman/snowman.py`
* The global variables at the top of the file and the `snowman` function in `snowman/snowman.py`
* All of the variables and functions in the `snowman/snowman.py` file
* The contents of both `main.py` and `snowman/snowman.py`
##### !end-options

##### !answer
* The `snowman` function in `snowman/snowman.py`
##### !end-answer

##### !explanation

We know the issue is with line 43 - `print(f"Sorry, you lose! The word was {snowman}")`. If we include only this line in the prompt, ChatGPT has no way of knowing what the variable `snowman` represents or where it’s declared and could make very wrong assumptions.

If we include the whole `snowman` function, ChatGPT will be able to see where `snowman` is declared and that it’s the name of the function, rather than a variable holding a string. Even though we don’t include the global variables, since we know from the problem description that the losing message always prints at the correct time, ChatGPT should still be able to give us helpful guidance about what is wrong with the message on line 43.

If we did not know for sure the bug was on line 43, or were investigating a different bug in this file, we would want to include much more context, including our global variables, likely the rest of the functions in `snowman/snowman.py` and possibly the contents of `main.py` as well. In general, if ChatGPT has an issue understanding the code that was shared or pinpointing the issue from the shared sample, you can always continue the conversation and provide more context.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->