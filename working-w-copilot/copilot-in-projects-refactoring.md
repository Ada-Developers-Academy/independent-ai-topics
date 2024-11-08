#  Copilot in Projects Pt. 2 - Refactoring

## Goals

We've used Copilot to generate new code in a project. Now we're going to see how Copilot can help us refactor existing code. Let's continue our trip back the [hello-books example project](https://github.com/AdaGold/hello-books-api) for a look at some additional ways we can use Copilot to increase our productivity.

Our goals for this lesson are to:
- identify where Copilot can assist us with refactoring
- get more experience adding and updating test cases with Copilot

## Refactoring with Copilot

Writing new code with Copilot is great, but a significant portion of software development involves refactoring and updating existing code. We got a little taste of updating code with the `Book` class, but we're going to revisit a scenario that we've seen before—refactoring the `validate_book` function into a more generic function that we can use for any model.

For this section of the lesson, we're going to check out a different `hello-books` branch, [`07b-to-dict-refactor`](https://github.com/AdaGold/hello-books-api/tree/07b-to-dict-refactor), as the starting point.

### !callout-info

## Saving Project Changes

If we forked the `hello-books` repo and want to keep any local changes that we made in the previous part of the lesson, we recommend pausing to create a new branch from the current one then pushing that code up before checking out the `07b-to-dict-refactor` branch. Git will not allow us to switch branches if we have uncommitted changes that would be overwritten by the destination branch.

### !end-callout

### !callout-info

## Running the Project and Tests

In this branch, the startup code in `app/__init__.py` has been updated to read its connection string from an environment variable. If we are working in a new cloned repo and don't have a `.env`, the following file can be used:

<br />

<details>
<summary><code>.env</code></summary>

```bash
SQLALCHEMY_DATABASE_URI=postgresql+psycopg2://postgres:postgres@localhost:5432/hello_books_development
SQLALCHEMY_TEST_DATABASE_URI=postgresql+psycopg2://postgres:postgres@localhost:5432/hello_books_test
```

</details>

<br />

Be sure that the `hello_books_development` and `hello_books_test` databases exist, and that the migrations have been run to create the tables in the development database. If there are already tables there, dropping and recreating the database will allow us to re-run the migrations if we're unsure of the current state of the database.

### !end-callout

### Planning the `validate_book` Refactor

Our steps to plan a refactor with Copilot stay the same: we first need to identify our dependencies, then check if we have tests for code that will be affected. We will be refactoring the `validate_book` function in the `book_routes.py` file, so we should search the project for where the function is used to remind ourselves of our dependencies.

This is a scenario where Copilot shouldn't be used, since Copilot has access to open files, but not all of the files in your project. Indeed, if we ask Copilot for help, it may warn us about this lack of access and give us suggestions on how to search the project for dependencies.

<br />

<details>
  <summary>
    If curious, feel free to try asking Copilot to identify dependencies for <code>validate_book</code> or expand this section to see our prompt and what Copilot suggested.
  </summary>

  **Prompt:**
  > What code in this project is dependent on the validate_book function?

  **Copilot's Response:**
  > As an AI, I don't have access to your entire project's codebase, so I can't directly identify all the places where `validate_book` function is used. However, typically, such a function would be used in routes that require a book ID as a parameter.
  >  
  > For example, you might have routes like `@books_bp.get("/<book_id>")`, `@books_bp.put("/<book_id>")`, or `@books_bp.delete("/<book_id>")` where you would use `validate_book(book_id)` to ensure the provided book ID is valid before proceeding with the GET, PUT, or DELETE operation.
  >  
  > Please search your codebase for `validate_book` to find all its usages. In Visual Studio Code, you can do this by pressing `Ctrl+Shift+F` and typing `validate_book` in the search bar.
</details>

Searching our project, we find 4 dependencies:
- the `validate_book` function definition
- usage in `read_one_book`
- usage in `update_book`
- usage in `delete_book`

Peeking at our test suite in `test_book_routes.py`, we do have tests for `read_one_book`, but we need to write tests for the remaining untested dependencies: `validate_book`, `update_book`, and `delete_book`.

### Updating Our Test Suite

#### Testing `validate_book`

We'll start with testing the `validate_book` function. Before we begin, we're going to open several files to ensure Copilot has plenty of context around our code and test set up. Take a moment to ensure the following files are open in VS Code:
- `book.py`
- `book_routes.py`
- `conftest.py`
- `test_book_model.py`
- `test_book_routes.py`

Start in `book_routes.py` and use the cursor to highlight the entire `validate_book` function. Next, bring up the inline Copilot chat and invoke the `/tests` shortcut. When Copilot is done thinking, we may see a slightly different UI than in previous scenarios.

This time, Copilot has opened a pane to the right with our existing `test_book_routes.py` file. At the end of the file's original content we have a Copilot chat box that we can interact with and below that are the tests that Copilot is suggesting to us.

![A VS Code window with a temporary file that contains the original contents of test_book_routes.py along with suggested changes to test the validate_book function](assets/copilot-in-projects/test-routes-validate-book-refactor-preview.png)  
*Fig. Temporary file in VS Code showing the test suggestions from Copilot ([Full size image](assets/copilot-in-projects/test-routes-validate-book-refactor-preview.png))*

Here we can review the test cases Copilot generated and see if there are changes or further tests we need. In the Copilot chat box we have "Apply", "Discard", and "Regenerate" controls to accept, reject, or ask Copilot to retry writing the tests, but we can also use the chat to ask for specific changes.

Examining the code created by Copilot, the scenarios identified are great: we have tests for the nominal case and a couple edge cases of invalid or non-existent book ids. However, there is a significant issue with all of the tests. Instead of testing the `validate_book` function by importing and invoking it directly, the tests are all making requests to the `GET` "`books/<book_id>`" endpoint. These tests aren't confirming that the `validate_book` function returns a specific value, they are testing a route which uses the function and which could have other side effects. Let's use the prompt below in the chat window to ask Copilot to rewrite these tests to invoke `validate_book` directly:

> Please rewrite the validate_book tests to import and directly invoke the validate_book function

Copilot's response is looking better, in our case Copilot added the import for `validate_book` at the top of the file and generated the following tests:
```py
from app.routes import validate_book

...

def test_validate_book_succeeds(client, two_saved_books):
    # Act
    book = validate_book(1)

    # Assert
    assert book.id == 1
    assert book.title == "Ocean Book"
    assert book.description == "watr 4evr"

def test_validate_book_missing_record(client, two_saved_books):
    # Act & Assert
    with pytest.raises(NotFound):
        validate_book(3)

def test_validate_book_invalid_id(client, two_saved_books):
    # Act & Assert
    with pytest.raises(BadRequest):
        validate_book("cat")
```

These tests are getting closer to what we need, but still won't work for us. The main issues are that:
- The `validate_book` import path at the top of the file is slightly wrong
- The tests require `pytest` to use `with pytest.raises`
- The symbols `NotFound` and `BadRequest` used in the edge case tests do not exist
- The status code isn't being checked on the edge cases where the request should have been aborted
- The `client` fixture is not required on any of the tests since we aren't making a request to an endpoint

Fixing the imports is a couple lines of code, but for our edge cases, we may need to do some research to be sure of the error we expect to be raised by our functions and ensure that our tests are looking for that particular error to be raised. In this case, we know from the Building an API series that the `validate_book` function will raise an `HTTPException` that we need to import from `werkzeug.exceptions` to access in our tests.

### !callout-info

## Be Sure to Review Your Own Output

Remember, the tests generated during the Copilot run in this lesson may not match what you see when trying on your own. Always closely review any suggestions from Copilot or any other AI tool to ensure they meet your needs and are correct for your project.

### !end-callout

Since we want these test scenarios, and the tests bodies are pretty close to what we're looking for, let's use the "Apply" button in the `Refactor Preview` tab to accept the changes. We'll address the issues listed above by:
- adjusting the imports 
- removing the client fixture from the tests' parameters
- changing the expected error in the `with pytest.raises` statements
- updating the assertions to check for a status where applicable

<br />

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

We'll start by using Copilot inline chat and the `/tests` shortcut to ask for help generating tests for the `update_book` function. Here's what Copilot suggested:
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

*Updated `test_update_book`*
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

The overall frame of the tests is looking good so we can press "Accept" and save the file. We could see about writing a fixture `db_session` like Copilot suggested, but we will choose to import `db` into the test file since we need to use both the `select` method and `session` attribute from `db` to query for a specific record. If we import `Book` and `db` and change our querying syntax over to using `db.select` and `db.session` our invalid and deprecated syntax issues are solved, but if we try to run the tests, we'll see a `JSONDecodeError`. This is because the line `response.get_json()` will raise a `JSONDecodeError` when the `response` object has an empty body. We'll address this last issue by either manually or with Copilot's help replacing the `response_body` assert with one that checks if `response.content_length is None`. 

*Final `test_update_book`*
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

Before we move on, there are a few other scenarios that are worth testing. What happens if the request dictionary is missing either the `"title"` or `"description"` keys? How will our code behave if the user adds extra keys to the dictionary in the request? We can use the Copilot chat box to ask for help generating the tests for these scenarios.

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