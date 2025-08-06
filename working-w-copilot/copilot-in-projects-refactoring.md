#  Copilot in Projects Pt. 2 - Best Practices in Wave 1

## Goals

We've used Copilot to generate new code in a project, but we saw examples where the code generated wasn't following best practices, especially around avoiding repetition. 

Our goals for this lesson are to identify where Copilot can assist us to better follow best practices, specifically:
- writing cleaner code with the the D.R.Y. (Don't Repeat Yourself) principle
- writing more Pythonic code by using patterns like list comprehensions 

### !callout-info

## Generative tools will give you different responses

As you work through this lesson, you will likely get different results from the prompts you submit than what we show through the lesson. This is expected!

<br>

We are working with a generative AI tool and they are not guaranteed to return the same or even a similar response for the same input. Part of adjusting to working with AI tools is getting comfortable with the variability of their responses and then fine tuning our prompts, regenerating responses, and manually updating generated code until we have something that meets our needs.

<br>

Even the Copilot extension itself is updated regularly, so the way the UI looks or where certain features are located may differ from that shown in this lesson. So if a screenshot looks a little different from what you see in your own VS Code, try to find the equivalent feature in your version of the extension.

### !end-callout

## Set Up & Tests

Writing new code with Copilot is great, but a significant portion of software development involves reviewing and updating existing code. We're going to revisit Wave 1 of `mastermind-copilot`, starting from a branch with Wave 1 completed. 
- We have the same test suite in `tests/test_wave_1.py` that we ended with in the prior lesson, but the function implementations in `app/game.py` are slightly different.

Before we make changes to working code, we want to make sure that we have tests for any code that could be affected by our updates. 
- Running the tests before and after we make changes gives us confidence that our functions still behave as expected. 
- If anything does go wrong, we can debug the functions using our tests to help us identify what broke. 
Happily, we not only have tests for each function in Wave 1, but we added more test scenarios in the last lesson to cover missing edge cases! We can move forward with our code changes using the existing test suite to give us some security.

As with the `main` branch, only the tests in `tests/test_wave_1.py` should be discoverable and running currently, since we do not have placeholders in `app/game.py` for the remaining waves. Before we start a conversation with Copilot or change any code, we should ensure that we can run all the Wave 1 tests and see them pass.

To get started:
1. using your fork of the repo, checkout the branch [improvements-start](https://github.com/Ada-Activities/mastermind-copilot/tree/improvements-start) 
2. ensure you can run the wave 1 tests and see them pass

### !callout-info

## Saving Project Changes

If we want to keep any local changes that we made in the previous lesson, we recommend pausing to create a new branch from `main` and commit your changes before checking out the `improvements-start` branch. Git will not allow us to switch branches if we have uncommitted changes that would be overwritten by the destination branch.

### !end-callout

## Identifying Code for Improvement

Let's review the implementations of the functions for Wave 1 on the `improvements-start` branch to get familiar with the changes and start looking for places where we could better follow best practices.

```py
import random


# Wave 1
def generate_code():
    valid_letters = ['R', 'O', 'Y', 'G', 'B', 'P']
    code = []
    
    # Generate a code of 4 random letters from valid_letters
    for _ in range(4):
        code.append(random.choice(valid_letters))

    return code


def validate_guess(guess):
    valid_letters = {'R', 'O', 'Y', 'G', 'B', 'P'}

    # Exit early if guess is not exactly 4 elements long
    if len(guess) != 4:
        return False
    
    # Convert guess to uppercase for case-insensitive comparison
    uppercased_guess = []
    for letter in guess:
        uppercased_guess.append(str(letter).upper())

    # Return False if we find an invalid element of guess
    for letter in uppercased_guess:
        if letter not in valid_letters:
            return False
        
    return True


def check_win_or_lose(guess, code, num_guesses):
    # Exit early if the number of guesses exceeds 8
    if num_guesses > 8:
        return False

    # Convert guess to uppercase for case-insensitive comparison
    uppercased_guess = []
    for letter in guess:
        uppercased_guess.append(str(letter).upper())

    # Check if the guess and code are identical (win condition)
    # The guard clause guarantees the number of guesses is 8 or less
    if code == uppercased_guess:
        return True
    else: # Game is still in progress
        return None
```

Taking a look at these implementations, they work as they are supposed to – our tests pass after all! – but there are areas where we could better follow best practices.

<br>

<details>
  <summary>
    Take a few minutes to examine the functions above, and identify areas of code that could be improved to better follow best practices or increase readability. Expand this section to see the issues we noted.
  </summary>

  **Areas for Improvement**
  1. `generate_code` and `validate_guess` require similar data about valid letters, but that data is duplicated in each function instead of being shared.
  2. `validate_guess` and `check_win_or_lose` duplicate work by creating uppercased versions of the input in each function without using a shared helper function.
  3. All of the functions create lists of data that do not require significant processing or data manipulation, but they are not using list comprehensions. List comprehensions are considered more pythonic, and better practice when working in Python 
  
  <br>

  We could choose to note one other potential area for improvement: 
  - `validate_guess` declares the `valid_letters` set before the guard clause that checks the length of `guess`, so the set is created even if it will never be used. 
  Since we already identified that we want `generate_code` and `validate_guess` to share a list of valid letters, we can handle this change during the updates to share the letter data. 

</details>

We will investigate and update the 3 issues we identified through this lesson. If you came up with more items than we listed, feel free to explore those as practice!

## D.R.Y. - Sharing Data Structures in `generate_code` and `validate_guess`

Once we see our Wave 1 tests passing using the current implementations of `generate_code` and `validate_guess`, we can move over to `app/game.py`. 

Before we send anything to Copilot, let's organize our information. We know:
- `generate_code` and `validate_guess` require similar data about valid letters
- we want to move the similar data structures out of each function and share them through something like a global variable

<br>

<details>
  <summary>
    Take a moment to write up a prompt that requests the changes for these functions. When you're done, expand this section to see the prompt we will use.
  </summary>

  **Prompt:**
  > Please update the generate_code and validate_guess functions to share a single data structure to represent valid_letters. The data structure should be a set type and should be placed where both functions have access to it. Do not make any other changes to the functions.

  <br>

  We specifically chose to add _"Do not make any other changes to the functions."_ to our prompt because in testing, we saw that Copilot would often try to make several changes to the function at the same time. This makes it less clear which changes are related to our initial ask vs what Copilot thought would improve the code. 
  - Changing code is an error-prone process. To avoid potential confusion and bugs, we want to focus on a single change at a time and see it successfully completed before updating another aspect of the code. 

</details>

Let's use the inline Copilot chat again, this time highlighting both `generate_code` and `validate_guess` then pressing `⌘I` (`CMD + i`) to bring up the Copilot textbox.

![VS Code with app/game.py open highlighting both functions generate_code and validate_guess. The inline Copilot text box is displayed above the functions with nothing entered.](assets/improving-code-copilot/dry-inline-chat-select-generate-code-and-validate-guess.png)  
*Fig. Inline Copilot chat opened with both `generate_code` and `validate_guess` highlighted ([Full size image](assets/improving-code-copilot/dry-inline-chat-select-generate-code-and-validate-guess.png))*

Once we submit our prompt, we have some changes to review. Let's take a closer look at the revised code from Copilot:

```py
VALID_LETTERS = {'R', 'O', 'Y', 'G', 'B', 'P'}

def generate_code():
    code = []
    
    # Generate a code of 4 random letters from VALID_LETTERS
    for _ in range(4):
        code.append(random.choice(list(VALID_LETTERS)))

    return code


def validate_guess(guess):
    # Exit early if guess is not exactly 4 elements long
    if len(guess) != 4:
        return False
    
    # Convert guess to uppercase for case-insensitive comparison
    uppercased_guess = []
    for letter in guess:
        uppercased_guess.append(str(letter).upper())

    # Return False if we find an invalid element of guess
    for letter in uppercased_guess:
        if letter not in VALID_LETTERS:
            return False
        
    return True
```

This is close to what we want! 
- We have a shared data structure that's a set like we wanted
- No unrelated sections of code were changed

As we look at this code, we need to ask ourselves, "Is this new code following best practices and acting efficiently?" Examining how the loops were changed, `validate_guess` looks fine, but how does Copilot's code use the shared data structure `VALID_LETTERS` in `generate_code`'s loop? 

There's something interesting happenning with the changes in `generate_code`. `random.choice` does not work with set data structures, so in the code Copilot generated, every iteration of the loop creates a new list from the `VALID_LETTERS` set. 
- The loop would be much more efficient if the function only created a list from `VALID_LETTERS` once before the loop starts. 

We could ask Copilot to make this change for us, but it would be quicker to accept the code as-is, then make this small adjustment ourselves. We'll do just that, then run our Wave 1 test suite to ensure both of our functions still work as expected.

Our final version of `generate_code` will look like:

```py
VALID_LETTERS = {'R', 'O', 'Y', 'G', 'B', 'P'}

def generate_code():
    code = []
    
    # Generate a code of 4 random letters from VALID_LETTERS
    letters_list = list(VALID_LETTERS)
    for _ in range(4):
        code.append(random.choice(letters_list))

    return code
```

## D.R.Y. - Helper Function for `validate_guess` and `check_win_or_lose`

<br>

<details>
  <summary>
    Take a moment to adjust the test file for best practices and to make sure all tests are passing. Feel free to expand this section when done to view the changes to our <code>test_book_routes.py</code> file.
  </summary>

  **Updated test_book_routes.py**
  ```py
  from app.routes.book_routes import validate_book
  from werkzeug.exceptions import HTTPException
  import pytest

  ...

  def test_validate_book_succeeds(two_saved_books):
    # Act
    book = validate_book(1)

    # Assert
    assert book.id == 1
    assert book.title == "Ocean Book"
    assert book.description == "watr 4evr"

  def test_validate_book_missing_record(two_saved_books):
    # Act & Assert
    with pytest.raises(HTTPException) as error:
        validate_book(3)

    response = error.value.response
    assert response.status == "404 NOT FOUND"

  def test_validate_book_invalid_id(two_saved_books):
    # Act & Assert
    with pytest.raises(HTTPException) as error:
        validate_book("cat")

    response = error.value.response
    assert response.status == "400 BAD REQUEST"
  ```
</details>

#### Testing `update_book`

With `validate_book` we were lucky, and Copilot outlined all of the scenarios that we intended to test, but that will not always be the case. We won't go step by step for each dependency in this lesson, but let's apply the same approach we took with `validate_model` to testing our remaining dependencies `update_book` and `delete_book`.

We'll start by using Copilot inline chat and the `/tests` shortcut to ask for help generating tests for the `update_book` endpoint. Here's what Copilot suggested:
```py
def test_update_book(client, two_saved_books):
    # Act
    response = client.put("/books/1", json={
        "title": "Updated Ocean Book",
        "description": "Updated description"
    })
    response_body = response.get_json()

    # Assert
    assert response.status_code == 204
    assert response_body is None

    # Verify the update
    response = client.get("/books/1")
    response_body = response.get_json()
    assert response_body == {
        "id": 1,
        "title": "Updated Ocean Book",
        "description": "Updated description"
    }

def test_update_book_missing_record(client, two_saved_books):
    # Act
    response = client.put("/books/3", json={
        "title": "Updated Book",
        "description": "Updated description"
    })
    response_body = response.get_json()

    # Assert
    assert response.status_code == 404
    assert response_body == {"message": "book 3 not found"}

def test_update_book_invalid_id(client, two_saved_books):
    # Act
    response = client.put("/books/cat", json={
        "title": "Updated Book",
        "description": "Updated description"
    })
    response_body = response.get_json()

    # Assert
    assert response.status_code == 400
    assert response_body == {"message": "book cat invalid"}
```

We already have a good range of scenarios:
- successfully update a book
- return an error if the book id is invalid
- return an error if a book with the id doesn't exist

Something to note is that the `test_update_book` test is using our `GET` `books/<book_id>` route to verify that the database changed. Instead of relying on another route, we could check the database itself. We can ask Copilot to make this change with a prompt like:

> Please update these tests to verify changes using the database rather than client.get

*Updated `test_update_book`:*
```py
def test_update_book(client, two_saved_books, db_session):
    # Act
    response = client.put("/books/1", json={
        "title": "Updated Ocean Book",
        "description": "Updated description"
    })
    response_body = response.get_json()

    # Assert
    assert response.status_code == 204
    assert response_body is None

    # Verify the update using the database
    updated_book = db_session.query(Book).get(1)
    assert updated_book.title == "Updated Ocean Book"
    assert updated_book.description == "Updated description"
```

We are using the database now, but from these new changes we need to address that:
- we must import the `Book` class
- the test is using a fixture `db_session` we haven't created
- Copilot chose deprecated querying syntax `db_session.query(Book).get(1)`. 

The overall frame of the tests is looking good so we can press "Accept" and save the file. We could see about writing a fixture `db_session` like Copilot suggested, but we will choose to import `db` into the test file since we need to use both the `select` method and `session` attribute from `db` to query for a specific record. 

If we import `Book` and `db` and change our querying syntax over to using `db.select` and `db.session`, our invalid and deprecated syntax issues are solved, but if we try to run the tests, we'll see a `JSONDecodeError`. This is because the line `response.get_json()` will raise a `JSONDecodeError` when the `response` object has an empty body. We'll address this last issue by, either manually or with Copilot's help, replacing the `response_body` assert with one that checks if `response.content_length is None`. 

*Final `test_update_book`:*
```py
def test_update_book(client, two_saved_books):
    # Act
    response = client.put("/books/1", json={
        "title": "Updated Ocean Book",
        "description": "Updated description"
    })

    # Assert
    assert response.status_code == 204
    assert response.content_length is None

    # Verify the update using the database
    query = db.select(Book).where(Book.id == 1)
    updated_book = db.session.scalar(query)
    assert updated_book.title == "Updated Ocean Book"
    assert updated_book.description == "Updated description"
```

Before we move on, there are a few other scenarios that are worth testing. What happens if the request dictionary is missing either the `"title"` or `"description"` keys? How will our code behave if the user adds extra keys to the dictionary in the request? We can open the Copilot chat in our test file to ask for help generating the tests for these scenarios.

<br />

<details>
  <summary>
    Take a moment to write a prompt for Copilot to generate tests for the missing scenarios mentioned above, then expand this section to see the prompt we used.
  </summary>

  **Prompt:**
  > Please write 3 new tests for the update_book function that cover the scenarios:
  > - Where the request dictionary has extra keys other than the required ones
  > - Where the request dictionary is missing the title
  > - Where the request dictionary is missing the description
</details>

When we asked for help with testing these scenarios, for the first test `test_update_book_with_extra_keys` Copilot gave us a suggestion with expectations that looked a little different than what we had set up in our prior `update_book` tests:
```py
def test_update_book_with_extra_keys(client, two_saved_books):
    # Act
    response = client.put("/books/1", json={
        "title": "Updated Title",
        "description": "Updated description",
        "extra_key": "extra_value"
    })
    response_body = response.get_json()

    # Assert
    assert response.status_code == 200
    assert response_body["title"] == "Updated Title"
    assert response_body["description"] == "Updated description"

def test_update_book_missing_title(client, two_saved_books):
    # Act
    response = client.put("/books/1", json={
        "description": "Updated description"
    })
    response_body = response.get_json()

    # Assert
    assert response.status_code == 400
    assert response_body == {"message": "Missing required field: title"}

def test_update_book_missing_description(client, two_saved_books):
    # Act
    response = client.put("/books/1", json={
        "title": "Updated Title"
    })
    response_body = response.get_json()

    # Assert
    assert response.status_code == 400
    assert response_body == {"message": "Missing required field: description"}
```

We could manually update the `test_update_book_with_extra_keys` test to match better, but Copilot can also reformat it to be closer to what we're looking for if we follow up with a prompt like:

> Please follow the patterns of the prior tests to validate changes and respect the expected response body

The resulting test code Copilot suggests still has issues we will address, but `test_update_book_with_extra_keys` now follows the same patterns as our other tests:

```py
def test_update_book_with_extra_keys(client, two_saved_books):
    # Act
    response = client.put("/books/1", json={
        "title": "Updated Ocean Book",
        "description": "Updated description",
        "extra_key": "extra_value"
    })

    # Assert
    assert response.status_code == 204
    assert response.content_length is None

    # Verify the update using the database
    query = db.select(Book).where(Book.id == 1)
    updated_book = db.session.scalar(query)
    assert updated_book.title == "Updated Ocean Book"
    assert updated_book.description == "Updated description"

def test_update_book_missing_title(client, two_saved_books):
    # Act
    response = client.put("/books/1", json={
        "description": "Updated description"
    })
    response_body = response.get_json()

    # Assert
    assert response.status_code == 400
    assert response_body == {"message": "Missing required field: title"}

def test_update_book_missing_description(client, two_saved_books):
    # Act
    response = client.put("/books/1", json={
        "title": "Updated Ocean Book"
    })
    response_body = response.get_json()

    # Assert
    assert response.status_code == 400
    assert response_body == {"message": "Missing required field: description"}
```

Let's "Accept" and save these tests then run them. The test `test_update_book_with_extra_keys` should pass as-is, but the other two are failing. We need to take a deeper look at the tests for missing required data.

The assertions for these tests assume that a 400 status code will be sent in case of an error, and also made an assumption about what the error message in the response would be. If we navigate to the `update_book` function in `book_routes.py`, we see that there actually isn't any error handling for this scenario. These tests fail because our application crashes with a `KeyError` when required data is missing from the request dictionary.

While it's possible that Copilot could have written tests that expected a crash (which would look like a `500 Internal Server Error`), it's much more common that these scenarios would result in a `400 Bad Request`. So even though Copilot generated failing tests, it turns out that this helped us by highlighting error handling that would be useful to have. Rather than changing the tests to expect a crash, we're going to add error handling to `update_book` to make our app more robust, then update our test assertions to check for the correct message.

<br />

<details>
  <summary>
    Try out adding error handling and updating the test assertions, then expand this section to see how we changed <code>update_book</code>, <code>test_update_book_missing_title</code>, and <code>test_update_book_missing_description</code>.
  </summary>

  **Updated `update_book` function:**
  ```py
  @books_bp.put("/<book_id>")
  def update_book(book_id):
    book = validate_book(book_id)
    request_body = request.get_json()

    try:
        book.title = request_body["title"]
        book.description = request_body["description"]

    except KeyError as e:
        abort(make_response({"message": f"{e.args[0]} is required"}, 400))

    db.session.commit()
    return Response(status=204, mimetype="application/json") # 204 No Content
  ```

  **Updated tests:**
  ```py
  def test_update_book_missing_title(client, two_saved_books):
    # Act
    response = client.put("/books/1", json={
        "description": "Updated description"
    })
    response_body = response.get_json()

    # Assert
    assert response.status_code == 400
    assert response_body == {"message": "title is required"}

  def test_update_book_missing_description(client, two_saved_books):
    # Act
    response = client.put("/books/1", json={
        "title": "Updated Ocean Book"
    })
    response_body = response.get_json()

    # Assert
    assert response.status_code == 400
    assert response_body == {"message": "description is required"}
  ```
</details>

Now that we've seen several facets of working with Copilot to write tests, we will leave it as a personal exercise to complete the test suite for our last untested dependency `delete_book`.

#### Executing the Refactor

Now that we have a solid test suite, we can get some help from Copilot to make this refactor go a little more quickly. Let's review what needs to change in `validate_book` to make the function more flexible so we can use it with other models:
- take in a second parameter that represents a model class reference
- change any uses of the `Book` class to use our new parameter
- update the function name, variable names, and return messages so they reflect that the function is not specific to the `Book` class

The changes we need are pretty minimal and could be quick to perform manually, but for practice, let's see how Copilot handles the change. Let's highlight the `validate_book` function, open up the inline Copilot chat, and provide the following prompt:

> Please update this function so that we could use it for any model class

The response delivers exactly what we're looking for:
```py
def validate_model(model_class, model_id):
    try:
        model_id = int(model_id)
    except ValueError:
        response = {"message": f"{model_class.__name__.lower()} {model_id} invalid"}
        abort(make_response(response, 400))

    query = db.select(model_class).where(model_class.id == model_id)
    model_instance = db.session.scalar(query)

    if not model_instance:
        response = {"message": f"{model_class.__name__.lower()} {model_id} not found"}
        abort(make_response(response, 404))

    return model_instance
```

Our function is updated, now we just need to update our dependent functions and tests to use `validate_model` instead of `validate_book`, which includes passing in the new `model_class` parameter. We may also decide to rename `model_class` to the more common `cls`, which is often used in Python when passing in a class reference. We can do all of this manually using VS Code's Find & Replace tools followed by some minor edits to add our new parameter, or we can reach out to Copilot.

To rename the class reference parameter, we select the `validate_model` function again, open the inline Copilot chat, and provide the following prompt:

> Please use cls for the class parameter name

Copilot can handle this request without issue. However, Find & Replace could do this just as easily, and we might even have more confidence in the results by performing the change manually. As usual, we need to review Copilot's changes carefully to ensure nothing was missed. In contrast, changing the rest of the code to use the refactored function is a more complex change that requires several edits, and we might not feel as confident jumping in on our own. This is a great opportunity to have Copilot lend a virtual hand, so let's have Copilot help us with the rest of the change.

One approach we can try is to open each file, select the entire contents, and ask Copilot to change any instances of `validate_book` to `validate_model`. We need to carefully review the lines Copilot suggests changing to ensure nothing unexpected is altered, but it can save us a little time since we need to do slightly more than just find and replace the function name.

Due to differences in context, it's possible that a prompt that works fine in one file may not work as well in another file. For example, in the `book_routes.py` file, the following prompt, which provides very few details, is able to produce the desired changes:

> Please replace all uses of validate_book with validate_model

However, in the `test_book_routes.py` file, more detail is necessary for Copilot to recognize that there is a new class parameter which is required wherever the function is invoked, and that the `Book` class needs to be imported:

> Please replace all uses of validate_book(book_id) with validate_model(cls, model_id). Import any required model classes.

Since there is always a degree of validation and review required with any change when using Copilot, if we're having trouble finding an effective prompt, it may be worth doing the refactoring some other way. We can always refactor by hand, by using standalone refactoring tools, or by using tools in our development environment. Copilot is just one tool among many that we can draw on as developers.

## Summary

This hands-on experience in a project wraps up our whirlwind tour of GitHub's Copilot! Whether we're writing new code or working in an existing codebase, Copilot can help us with tasks like generating ideas for tests, surfacing error handling we may have missed, and generally writing code more quickly. Even when Copilot's suggestions look good at a glance, we always need to carefully proofread them both for correctness and completeness. Just because a code suggestion works does not mean it fits all of our needs, so we need to remain vigilant and critical of what Copilot provides.

For more practice with Copilot, think about revisiting Solar System, Task List, Inspiration Board, or any other familiar project. When we already have a reasonable idea what the end product could look like, that lets us focus on practicing new skills, such as working with Copilot.

## Check for Understanding

<!-- prettier-ignore-start -->
### !challenge
* type: checkbox
* id: b53afd64-55e1-4670-8d3b-bce1ca3b8872
* title: Copilot in Projects Pt. 2 - Refactoring

##### !question
Select the refactoring steps below where Copilot may **not** be useful.
##### !end-question

##### !options
a| Gathering requirements
b| Locating Dependencies
c| Checking and expanding our test suite
d| Applying the code change
##### !end-options

##### !answer
a|
b|
##### !end-answer

##### !hint
- Can Copilot read our minds and know what we want to build?
- Does Copilot have access to every file in a project at once?
- Can Copilot examine test files and application code and suggest new tests?
- Can Copilot apply refactoring changes for us?
##### !end-hint

##### !explanation
* Gathering requirements - Copilot has no way of knowing what it is we want to build and requirements like speed, memory, etc.; those need to be decided by people, ideally before we start building the software.
* Locating Dependencies - Copilot doesn't automatically have access to every file in a project, so it can easily miss dependencies.
##### !end-explanation

### !end-challenge
<!-- prettier-ignore-end -->