# Debugging with ChatGPT

## Goals

We have a number of tools in our belt for debugging and understanding code, but there’s always room for more! This lesson is about what we can do with LLMs and generative AI tools like ChatGPT if we’ve used all other debugging tools at our disposal and are still feeling stuck or unsure of what a piece of code is doing.

Our goals for this lesson are to:
- Learn how to effectively share code and context with ChatGPT
- Use ChatGPT in conjunction with our other debugging skills 
- Examine how to evaluate ChatGPT provided explanations for correctness

### !callout-warning

## Caveats & Concerns

First and foremost, we want to note that all of our debugging skills that we learned before are still necessary and helpful, even with generative AI tools. In many instances, we’ll be able to find and fix a bug faster by reading through the code or adding a couple break points than it would take to craft and iterate on a prompt to get an explanation. Furthermore, because tools like ChatGPT can fabricate information and are not running code themselves to ensure what they suggest works, we need to keep up our debugging and reasoning skills to evaluate what ChatGPT shares with us to see if it 1) makes sense and 2) truly addresses the initial problem.  
<br>
Second, and very importantly: depending on what it is you’re working on, you may not be allowed to send the code to ChatGPT or other generative AI tools. Enterprise solutions are being created and improved that allow companies to feed their proprietary code into an LLM-based tool without the code being used for further training or made in some way available to other people, but this is not the case for most publicly available LLMs and generative AI tools at this time.  
<br>
Be very careful about what you share with LLMs - an example of code that you could submit to ChatGPT to receive an explanation of concepts or debug with would be a personal project that you have complete control over. Any project which you don’t have full ownership over should be discussed and approved by other stakeholders before code is shared with generative AI tools of any kind.  

### !end-callout

## Tips for working with code in LLMs

### Setting the Stage

It’s helpful to set the stage first when working with LLM tools like ChatGPT while coding. We can do this by prompting ChatGPT with a role to "act as an expert in a particular language, helping you to debug". If we know exactly what problem we need help with we can include more details in the prompt, but a starting prompt could look like:

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

The type of information we’re gathering may sound familiar. When we ask ChatGPT for help, we need to collect the same kinds of detailed information and explanation that we would share when asking someone for help who may be unfamiliar with the project. 

## Providing Code & Enumerating Lines

### Sharing the “right” code

Many tools like ChatGPT 3.5 don’t have access to outside sources like GitHub to view code, so we need to be prepared to paste the relevant code into our prompt. We need to provide ChatGPT with all the context it requires to understand where the bug is happening, including details like global variables and imports. 

Let’s take a look at an example, say we have the following files in a project about dog sitting.

```py
# main.py
from dog_actions import greet_all_dogs


DOGS = [
    {
        "name": "Bud",
        "age": 1,
    },
    {
        "name": "Gouda",
        "age": 11,
    },    
    {
        "name": "Felix",
        "age": 3,
    },
    {
        "name": "Penny",
        "age": 1,
    }
]


def main():
    mode = input("Enter 't' for testing or 'r' to run: ")
    if mode.lower() == "t":
        new_dog = {
            "name": "Buttercup",
            "age": 6
        }
        result = greet_all_dogs([new_dog])
        expected = "You cautiously let Buttercup sniff your hand. You pet Buttercup gently. You wave goodbye to the dog, Buttercup. "
        assert result == expected

    else:
        print(greet_all_dogs(DOGS))

if __name__ == "__main__":
    main()
```

```py
# dog_actions.py
def let_dog_sniff(dog):
    dog_name = dog["name"]
    return f"You cautiously let {dog_name} sniff your hand."

def pet_dog(dog):
    dog_name = dog["name"]
    return f"You pet {dog_name} gently."

def say_bye_to_dog(dog):
    dog_name = dog["name"]
    return f"You wave goodbye to the dog, {dog_name}."

def greet_dog(dog):
    message = f"{let_dog_sniff(dog)} {pet_dog(dog)} {say_bye_to_dog(dog)}"
    return message

def greet_all_dogs(dog_list):
    all_messages = ""
    for dog in range(len(dog_list)):
        all_messages += f"{greet_dog(dog)} \n"

    return all_messages
```


We would like assistance from ChatGPT to debug why we're getting a crash when we run the `main` file. We could paste just the `main` function into our prompt, but `main` relies on on the imported function `greet_all_dogs` from `dog_actions.py`. Unless we share the `dog_actions.py` code, tools like ChatGPT would not know about it. 

In this case, by only including the `main` function, we severely limit the tool’s ability to understand the full picture. If we want to debug what is happening in the `main` function, then we should include the `main` function definition, as well as the entire `dog_actions.py` file contents in our prompt. 

### Enumerating Lines

To make it easy for ourselves and ChatGPT to refer to specific lines of the code, we can prompt ChatGPT to enumerate the lines of the code we just gave it. This way, we can refer to those lines of code by number in later prompts.

> Please enumerate the lines of the code I just gave you.

<details>
   <summary>Try out the prompt above with the <code>main.py</code> and <code>dog_actions.py</code> files, then expand this section to see the prompts we used to share the code and enumerate it along with ChatGPT's responses.</summary> 

Our Prompt:
```
Given this Python code:
'''
from dog_actions import greet_all_dogs


DOGS = [
    {
        "name": "Bud",
        "age": 1,
    },
    {
        "name": "Gouda",
        "age": 11,
    },    
    {
        "name": "Felix",
        "age": 3,
    },
    {
        "name": "Penny",
        "age": 1,
    }
]


def main():
    mode = input("Enter 't' for testing or 'r' to run: ")
    if mode.lower() == "t":
        new_dog = {
            "name": "Buttercup",
            "age": 6
        }
        result = greet_all_dogs([new_dog])
        expected = "You cautiously let Buttercup sniff your hand. You pet Buttercup gently. You wave goodbye to the dog, Buttercup. "
        assert result == expected

    else:
        print(greet_all_dogs(DOGS))

if __name__ == "__main__":
    main()
'''
'''
def let_dog_sniff(dog):
    dog_name = dog["name"]
    return f"You cautiously let {dog_name} sniff your hand."

def pet_dog(dog):
    dog_name = dog["name"]
    return f"You pet {dog_name} gently."

def say_bye_to_dog(dog):
    dog_name = dog["name"]
    return f"You wave goodbye to the dog, {dog_name}."

def greet_dog(dog):
    message = f"{let_dog_sniff(dog)} {pet_dog(dog)} {say_bye_to_dog(dog)}"
    return message

def greet_all_dogs(dog_list):
    all_messages = ""
    for dog in range(len(dog_list)):
        all_messages += f"{greet_dog(dog)} \n"

    return all_messages
'''
Respond with YES if you understand the code.
```

Chat GPT's response:
> YES. I understand the code.

Our Prompt:
```
Please enumerate the lines of the code I shared
```

Chat GPT's response:
> Sure! Here’s the code you shared with line numbers added:
>
> ```python
> 1  from dog_actions import greet_all_dogs
> 2  
> 3  
> 4  DOGS = [
> 5      {
> 6          "name": "Bud",
> 7          "age": 1,
> 8      },
> 9      {
> 10         "name": "Gouda",
> 11         "age": 11,
> 12     },    
> 13     {
> 14         "name": "Felix",
> 15         "age": 3,
> 16     },
> 17     {
> 18         "name": "Penny",
> 19         "age": 1,
> 20     }
> 21 ]
> 22 
> 23 
> 24 def main():
> 25     mode = input("Enter 't' for testing or 'r' to run: ")
> 26     if mode.lower() == "t":
> 27         new_dog = {
> 28             "name": "Buttercup",
> 29             "age": 6
> 30         }
> 31         result = greet_all_dogs([new_dog])
> 32         expected = "You cautiously let Buttercup sniff your hand. You pet Buttercup gently. You wave goodbye to the dog, Buttercup. "
> 33         assert result == expected
> 34 
> 35     else:
> 36         print(greet_all_dogs(DOGS))
> 37 
> 38 if __name__ == "__main__":
> 39     main()
> ```
> 
> ```python
> 1  def let_dog_sniff(dog):
> 2      dog_name = dog["name"]
> 3      return f"You cautiously let {dog_name} sniff your hand."
> 4  
> 5  def pet_dog(dog):
> 6      dog_name = dog["name"]
> 7      return f"You pet {dog_name} gently."
> 8  
> 9  def say_bye_to_dog(dog):
> 10     dog_name = dog["name"]
> 11     return f"You wave goodbye to the dog, {dog_name}."
> 12 
> 13 def greet_dog(dog):
> 14     message = f"{let_dog_sniff(dog)} {pet_dog(dog)} {say_bye_to_dog(dog)}"
> 15     return message
> 16 
> 17 def greet_all_dogs(dog_list):
> 18     all_messages = ""
> 19     for dog in range(len(dog_list)):
> 20         all_messages += f"{greet_dog(dog)} \n"
> 21 
> 22     return all_messages
> ```
> 
> Let me know how you’d like to proceed!

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

Let’s say that a student, Althea, is working on a game with a function that draws 7 random domino tiles from a pool. They are debugging an `IndexError` that sometimes appears in their `draw_tiles` function:

```
def draw_tiles():
    HAND_SIZE = 7
    pool = [
        "[0|0]", "[0|1]", "[0|2]", "[0|3]", "[0|4]", "[0|5]", "[0|6]", 
        "[1|1]", "[1|2]", "[1|3]", "[1|4]", "[1|5]", "[1|6]", "[2|2]",
        "[2|3]", "[2|4]", "[2|5]", "[2|6]", "[3|3]", "[3|4]", "[3|5]", 
        "[3|6]", "[4|4]", "[4|5]", "[4|6]", "[5|5]", "[5|6]", "[6|6]"
    ]
    drawn_tiles = []

    for _ in range(HAND_SIZE):
        index = random.randint(0, len(pool))
        tile = pool[index]
        drawn_tiles.append(tile)
        pool.remove(tile)

    return drawn_tiles
```

Althea has already taken steps to add print statements and step through their code with a debugger, and they are pretty sure the issue is somewhere in the for-loop. 

Let’s take a look at Althea’s conversation with ChatGPT and see how they used it for guidance while confirming the facts of the response:

> **Althea**:  
> You are a python expert who will help me debug python code. 

> **ChatGPT**:  
> Got it! Send over your Python code, and I'll help you debug it.

> **Althea**:  
> I am facing an issue with a piece of code. The function should always successfully return a hand of seven tiles. In reality, I am seeing a bug where I am receiving an "IndexError" sometimes, but not every time. I think the bug is in the for loop, what are possible causes?
>
> The code with the issue is pasted below. Please describe what the code does and enumerate the lines of code below as part of your response.
> 
> ```   
> def draw_tiles():
>    HAND_SIZE = 7
>    pool = [
>        "[0|0]", "[0|1]", "[0|2]", "[0|3]", "[0|4]", "[0|5]", "[0|6]", 
>        "[1|1]", "[1|2]", "[1|3]", "[1|4]", "[1|5]", "[1|6]", "[2|2]",
>        "[2|3]", "[2|4]", "[2|5]", "[2|6]", "[3|3]", "[3|4]", "[3|5]", 
>        "[3|6]", "[4|4]", "[4|5]", "[4|6]", "[5|5]", "[5|6]", "[6|6]"
>    ]
>    drawn_tiles = []
>
>    for _ in range(HAND_SIZE):
>        index = random.randint(0, len(pool))
>        tile = pool[index]
>        drawn_tiles.append(tile)
>        pool.remove(tile)
>
>    return drawn_tiles
> ``` 

> **ChatGPT**:  
> 1. `def draw_tiles():`
> - This defines the function draw_tiles with no parameters.
> 2. `HAND_SIZE = 7`
> - Defines a constant HAND_SIZE to specify that the hand should always contain 7 tiles.
> 3. `pool = [...]`
> - Initializes a list called pool that contains 28 tiles (each tile represented as a string with a pair of numbers like `"[0|0]"`, `"[1|2]"`, etc.).
> 4. `drawn_tiles = []`
> - Initializes an empty list drawn_tiles to store the tiles that are drawn.
> 5. `for _ in range(HAND_SIZE):`
> - A loop that will iterate HAND_SIZE (7) times. Each iteration represents one tile being drawn.
> 6. `index = random.randint(0, len(pool))`
> - This line generates a random integer index between 0 and the length of the pool list. This is where the bug happens. The issue is that random.randint(0, len(pool)) can generate an index equal to len(pool), which is an invalid index since list indices are 0-based (i.e., valid indices are from 0 to len(pool) - 1).
> 7. `tile = pool[index]`
> - The tile at the randomly chosen index is retrieved from the pool.
> 8. `drawn_tiles.append(tile)`
> - The drawn tile is ap`pended to the drawn_tiles list.
> 9. `pool.remove(tile)`
> - The drawn tile is removed from the pool to ensure it cannot be drawn again.
> 10. `return drawn_tiles`
> - After drawing seven tiles, the list drawn_tiles is returned, which contains the tiles drawn from the pool.
> ### Root Cause of the Bug:
> The issue lies in the line:
> ```
> index = random.randint(0, len(pool))
> ```
> Here, `random.randint(0, len(pool))` can return `len(pool)`, which is out of range for the list `pool` since valid indices for a list are from `0` to `len(pool) - 1`. When `len(pool)` is selected as the index, an `IndexError` will occur when trying to access `pool[index]`.
> ### Solution:
> To fix this, change the `random.randint` range to:
> ```
> index = random.randint(0, len(pool) - 1)
> ```
> This will ensure that the index is always within valid bounds.
> ### Revised Code:
> ```
> def draw_tiles():
>    HAND_SIZE = 7
>    pool = [
>        "[0|0]", "[0|1]", "[0|2]", "[0|3]", "[0|4]", "[0|5]", "[0|6]", 
>        "[1|1]", "[1|2]", "[1|3]", "[1|4]", "[1|5]", "[1|6]", "[2|2]",
>        "[2|3]", "[2|4]", "[2|5]", "[2|6]", "[3|3]", "[3|4]", "[3|5]", 
>        "[3|6]", "[4|4]", "[4|5]", "[4|6]", "[5|5]", "[5|6]", "[6|6]"
>    ]
>    drawn_tiles = []
>
>    for _ in range(HAND_SIZE):
>        index = random.randint(0, len(pool) - 1)  # Fix: len(pool) - 1
>        tile = pool[index]
>        drawn_tiles.append(tile)
>        pool.remove(tile)
>
>    return drawn_tiles
> ```
> This change should prevent the IndexError from occurring.

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
> Do you have suggestions for how I can simplify the code for choosing seven random tiles from the list `pool`?

> *ChatGPT*:  
> Certainly! You can simplify the code for choosing seven random tiles from the `pool` list using the `random.sample` function. The `random.sample` function is designed for this purpose and ensures that you get unique elements from the population.
>
> Simplified Version:
> 
> ```python
>import random
>
> def draw_tiles():
>    HAND_SIZE = 7
>    pool = [
>        "[0|0]", "[0|1]", "[0|2]", "[0|3]", "[0|4]", "[0|5]", "[0|6]", 
>        "[1|1]", "[1|2]", "[1|3]", "[1|4]", "[1|5]", "[1|6]", "[2|2]",
>        "[2|3]", "[2|4]", "[2|5]", "[2|6]", "[3|3]", "[3|4]", "[3|5]", 
>        "[3|6]", "[4|4]", "[4|5]", "[4|6]", "[5|5]", "[5|6]", "[6|6]"
>    ]
>    
>    return random.sample(pool, HAND_SIZE)  # Select 7 unique tiles
> ```
> 
> This code uses `random.sample(pool, HAND_SIZE)` to directly obtain a list of 7 unique elements from the `pool` list, eliminating the need for the loop and manual removal of elements.

We can see that ChatGPT is suggesting that Althea uses a separate function `sample` from the `random` module. Before copy/pasting the suggestion into their project, they go back to the random module documentation or other trusted resources to ensure: 
- Does the `random.sample` function exist? 
- If `random.sample` does exist, does it actually do what ChatGPT says it does? 
- If it does what ChatGPT thinks, then does using the function meet the original code requirements?

In this case, the answer to all of these questions is Yes! Althea can once again update their code and run the test suite to ensure everything is still in working order after their changes 🎉.

## Summary

ChatGPT can be a really helpful tool to give us more insight on what code is doing, but we need to be careful about over-reliance. There are many situations in life such as interviews and code review meetings where we may not have access to generative AI tools, meaning that tools like ChatGPT are best used to support and augment the debugging skills we’ve built rather than to replace them. When we start out a debugging journey, we still want to begin with searching errors on the web, using print statements, setting breakpoints to step through our code, and asking others for help before turning to ChatGPT. 

If we have trouble finding resources about the error or understanding what the code is doing using our prior skills, we can begin to augment our investigation by folding in ChatGPT to see if it can explain an error and common causes, or talk through what particular lines of code are doing in reality vs expectation. 

Before applying any suggestions from generative AI tools we need to break out our information validation skills to check if what they are suggesting is accurate for the language we’re working in and addresses the intended issue. And just like any situation where we’re bug fixing or refactoring, we should be running our tests frequently and expanding our test suite if necessary to ensure that we are truly addressing the issue without causing side effects.

The more we work with code, the more familiar we will be with common edge cases and coming up with the edge cases unique to our project, but we can also ask ChatGPT to help us brainstorm test cases to strengthen our test suites. 

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