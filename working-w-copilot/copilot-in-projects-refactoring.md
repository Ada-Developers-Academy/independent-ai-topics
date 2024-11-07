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

This is a scenario where Copilot shouldn't be used, since Copilot has access to open files, but not all of the files in your project. Indeed, if we ask Copilot for help, it will warn us about this lack of access and give us suggestions on how to search the project for dependencies.

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
  > For example, you might have routes like `@books_bp.route("/<book_id>", methods=["GET", "PUT", "DELETE"])` where you would use `validate_book(book_id)` to ensure the provided book ID is valid before proceeding with the GET, PUT, or DELETE operation.
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

Start in `book_routes.py` and use the cursor to highlight the entire `validate_book` function. Next, bring up the inline Copilot chat and invoke the `/tests` shortcut. When Copilot is done thinking, we may see a slightly different UI than in previous scenarios:

![A VS Code window with the routes.py file up and validate_book function selected showing a Refactor Preview tab at the bottom of the screen](assets/copilot-in-projects/routes-test-validate-book.png)  
*Fig. Copilot's `Refactor Preview` tab showing at the bottom of VS Code*

This time, instead of showing changes to the current file in a temporary window, the bottom panel has opened to a `Refactor Preview` tab that has a single entry on it. If we click the entry, it will take us to a temporary file showing the changes Copilot is suggesting that we make to the `test_book_routes.py` file.

![A VS Code window with a temporary file showing that contains the original contents of test_routes.py along with suggested changes to test the validate_book function](assets/copilot-in-projects/test-routes-validate-book-refactor-preview.png)  
*Fig. Temporary file in VS Code showing the test suggestions from Copilot*

Here we can review the test cases Copilot generated and see if there are changes or further tests we need. We have "Apply" and "Discard" controls at the bottom of the screen in the `Refactor Preview` tab to either accept or reject the changes.

Examining the code created by Copilot, the scenarios identified are great; we have tests for the nominal case and a couple edge cases of invalid or non-existent book ids. However, there are a number of issues:
- imports were added in the middle of the file, one of which is tacked on to the last line of an existing test
- `test_validate_book_with_valid_id` doesn't use our `two_saved_books` test fixture to ensure that there are books in the database before the test runs
- `test_validate_book_with_valid_id` thinks the result of `validate_book(book_id)` will be an integer rather than a `Book` instance
- `test_validate_book_with_invalid_id` and `test_validate_book_with_nonexistent_id` try to read the error value from outside the `with pytest.raises` block where it's out of scope

### !callout-info

## Be Sure to Review Your Own Output

Remember, the tests generated during the Copilot run in this lesson may not match what you see when trying on your own. Always closely review any suggestions from Copilot or any other AI tool to ensure they meet your needs and are correct for your project.

### !end-callout

Since we want these test scenarios, and the tests bodies are pretty close to what we're looking for, let's use the "Apply" button in the `Refactor Preview` tab to accept the changes. We'll address the issues listed above by adjusting the imports, adding the relevant test fixtures, and updating the assertions to ensure they are for the correct values and are located in scope of the objects they are trying to examine.

<br />

<details>
  <summary>
    Take a moment to adjust the test file for best practices and to make sure all tests are passing. Feel free to expand this section when done to view our updated <code>test_routes.py</code> file.
  </summary>

  **Updated test_routes.py**
  ```py
  import pytest
  from app.routes import validate_book

  def test_get_all_books_with_no_records(client):
      # Act
      response = client.get("/books")
      response_body = response.get_json()

      # Assert
      assert response.status_code == 200
      assert response_body == []

  def test_get_all_books_with_two_records(client, two_saved_books):
      # Act
      response = client.get("/books")
      response_body = response.get_json()

      # Assert
      assert response.status_code == 200
      assert len(response_body) == 2
      assert response_body[0] == {
          "id": 1,
          "title": "Ocean Book",
          "description": "watr 4evr"
      }
      assert response_body[1] == {
          "id": 2,
          "title": "Mountain Book",
          "description": "i luv 2 climb rocks"
      }

  def test_get_all_books_with_title_query_matching_none(client, two_saved_books):
      # Act
      data = {'title': 'Desert Book'}
      response = client.get("/books", query_string = data)
      response_body = response.get_json()

      # Assert
      assert response.status_code == 200
      assert response_body == []

  def test_get_all_books_with_title_query_matching_one(client, two_saved_books):
      # Act
      data = {'title': 'Ocean Book'}
      response = client.get("/books", query_string = data)
      response_body = response.get_json()

      # Assert
      assert response.status_code == 200
      assert len(response_body) == 1
      assert response_body[0] == {
          "id": 1,
          "title": "Ocean Book",
          "description": "watr 4evr"
      }

  def test_get_one_book_missing_record(client, two_saved_books):
      # Act
      response = client.get("/books/3")
      response_body = response.get_json()

      # Assert
      assert response.status_code == 404
      assert response_body == {"message":"book 3 not found"}

  def test_get_one_book_invalid_id(client, two_saved_books):
      # Act
      response = client.get("/books/cat")
      response_body = response.get_json()

      # Assert
      assert response.status_code == 400
      assert response_body == {"message":"book cat invalid"}

  def test_get_one_book(client, two_saved_books):
      # Act
      response = client.get("/books/1")
      response_body = response.get_json()

      # Assert
      assert response.status_code == 200
      assert response_body == {
          "id": 1,
          "title": "Ocean Book",
          "description": "watr 4evr"
      }

  def test_create_one_book(client):
      # Act
      response = client.post("/books", json={
          "title": "New Book",
          "description": "The Best!"
      })
      response_body = response.get_json()

      # Assert
      assert response.status_code == 201
      assert response_body == "Book New Book successfully created"

  def test_create_one_book_no_title(client):
      # Arrange
      test_data = {"description": "The Best!"}

      # Act & Assert
      with pytest.raises(KeyError, match='title'):
          response = client.post("/books", json=test_data)

  def test_create_one_book_no_description(client):
      # Arrange
      test_data = {"title": "New Book"}

      # Act & Assert
      with pytest.raises(KeyError, match = 'description'):
          response = client.post("/books", json=test_data)

  def test_create_one_book_with_extra_keys(client, two_saved_books):
      # Arrange
      test_data = {
          "extra": "some stuff",
          "title": "New Book",
          "description": "The Best!",
          "another": "last value"
      }

      # Act
      response = client.post("/books", json=test_data)
      response_body = response.get_json()

      # Assert
      assert response.status_code == 201
      assert response_body == "Book New Book successfully created"

  def test_validate_book_with_valid_id(two_saved_books):
      # Arrange
      book_id = 1

      # Act
      result = validate_book(book_id)

      # Assert
      assert result.id == book_id

  def test_validate_book_with_invalid_id():
      # Arrange
      book_id = "cat"

      # Act & Assert
      with pytest.raises(Exception) as e:
          validate_book(book_id)
          assert str(e.value) == "book cat invalid"

  def test_validate_book_with_nonexistent_id():
      # Arrange
      book_id = 3

      # Act & Assert
      with pytest.raises(Exception) as e:
          validate_book(book_id)
          assert str(e.value) == "book 3 not found"
  ```
</details>

#### Testing `update_book`

With `validate_book` we were lucky, and Copilot outlined all of the scenarios that we intended to test, but that will not always be the case. We won't go step by step again, but let's apply the same approach we took with `validate_model` to testing our remaining dependencies `update_book` and `delete_book`.

We'll start by using Copilot inline chat to ask for help generating tests for the `update_book` function. Here's what Copilot suggested:
```py
def test_update_book(client, two_saved_books):
    # Arrange
    book_id = 1
    data = {
        "title": "Updated Book",
        "description": "New description"
    }

    # Act
    response = client.put(f"/books/{book_id}", json=data)
    response_body = response.get_json()

    # Assert
    assert response.status_code == 200
    assert response_body == "Book #1 successfully updated"

def test_update_book_invalid_id(client, two_saved_books):
    # Arrange
    book_id = "cat"
    data = {
        "title": "Updated Book",
        "description": "New description"
    }

    # Act
    response = client.put(f"/books/{book_id}", json=data)
    response_body = response.get_json()

    # Assert
    assert response.status_code == 400
    assert response_body == {"message": "book cat invalid"}

def test_update_book_nonexistent_id(client, two_saved_books):
    # Arrange
    book_id = 3
    data = {
        "title": "Updated Book",
        "description": "New description"
    }

    # Act
    response = client.put(f"/books/{book_id}", json=data)
    response_body = response.get_json()

    # Assert
    assert response.status_code == 404
    assert response_body == {"message": "book 3 not found"}
```

This time Copilot did a great job writing tests that properly use our test fixtures and assert on the correct values! The tests above will run as is, and we already have a good range of scenarios:
- successfully update a book
- return an error if the book id is invalid
- return an error if a book with the id doesn't exist

However, there are other scenarios that are worth testing. What happens if the request dictionary is missing either the `"title"` or `"description"` keys? How will our code behave if the user adds extra keys to the dictionary in the request? We can open up Copilot chat and ask for help generating the tests for these scenarios.

<br />

<details>
  <summary>
    Take a moment to write a prompt for Copilot to generate tests for the missing scenarios mentioned above, then expand this section to see the prompt we used.
  </summary>

  **Prompt:**
  > Please write 3 more tests for the update_book function that cover the scenarios:
  > - where the request dictionary has extra keys other than the required ones
  > - Where the request dictionary is missing the title
  > - Where the request dictionary is missing the description
</details>

When we asked for help with testing these scenarios, Copilot gave us test suggestions, but they looked very different from our code in `test_book_routes.py`:
```py
import pytest
from flask import json

def test_update_book_with_extra_keys(client):
    response = client.put('/books/1', data=json.dumps({
        "title": "New Title",
        "description": "New Description",
        "extra_key": "Extra Value"
    }), content_type='application/json')

    assert response.status_code == 200
    assert b"Book #1 successfully updated" in response.data

def test_update_book_without_title(client):
    response = client.put('/books/1', data=json.dumps({
        "description": "New Description"
    }), content_type='application/json')

    assert response.status_code == 400
    assert b"Title is required" in response.data

def test_update_book_without_description(client):
    response = client.put('/books/1', data=json.dumps({
        "title": "New Title"
    }), content_type='application/json')

    assert response.status_code == 400
    assert b"Description is required" in response.data
```

We could manually update the tests to match better, but Copilot can also reformat them to be closer to what we're looking for if we follow up with a prompt like:

> Please update the tests above to match the formatting used in test_routes.py

The resulting test code Copilot suggests still has issues we will address, but it is structured much more closely to our other tests:

```py
def test_update_book_with_extra_keys(client, two_saved_books):
    # Arrange
    book_id = 1
    data = {
        "title": "Updated Book",
        "description": "New description",
        "extra_key": "Extra Value"
    }

    # Act
    response = client.put(f"/books/{book_id}", json=data)
    response_body = response.get_json()

    # Assert
    assert response.status_code == 200
    assert response_body == "Book #1 successfully updated"

def test_update_book_without_title(client, two_saved_books):
    # Arrange
    book_id = 1
    data = {
        "description": "New description"
    }

    # Act
    response = client.put(f"/books/{book_id}", json=data)

    # Assert
    assert response.status_code == 400
    assert b"Title is required" in response.data

def test_update_book_without_description(client, two_saved_books):
    # Arrange
    book_id = 1
    data = {
        "title": "Updated Book"
    }

    # Act
    response = client.put(f"/books/{book_id}", json=data)

    # Assert
    assert response.status_code == 400
    assert b"Description is required" in response.data
```

Let's paste these tests into our test file and run them. The test `test_update_book_with_extra_keys` passes without changes, but the other two are failing. We need to take a deeper look at the tests for missing required data.

The assertions for these tests assume that a 400 status code will be sent in case of an error, and also made an assumption about what the error message in the response would be. If we navigate to the `update_book` function in `book_routes.py`, we see that there actually isn't any error handling for this scenario. These tests fail because our application crashes with a `KeyError` when required data is missing from the request dictionary.

While it's possible that Copilot could have written tests that expected a crash (which would look like a `500 Internal Server Error`), it's much more common that these scenarios would result in a `400 Bad Request`. So even though Copilot generated failing tests, it turns out that this helped us by highlighting error handling that would be useful to have. Rather than changing the tests to expect a crash, we're going to add error handling to `update_book` to make our app more robust, then update our test assertions to check for the correct message.

<br />

<details>
  <summary>
    Try out adding error handling and updating the test assertions, then expand this section to see how we changed <code>update_book</code>, <code>test_update_book_without_title</code>, and <code>test_update_book_without_description</code>.
  </summary>

  **Updated `update_book` function:**
  ```py
  @books_bp.route("/<book_id>", methods=["PUT"])
  def update_book(book_id):
      book = validate_book(book_id)

      request_body = request.get_json()

      try:
          book.title = request_body["title"]
          book.description = request_body["description"]

      except KeyError as e:
          abort(make_response({"message": f"{e.args[0]} is required"}, 400))

      db.session.commit()

      return make_response(jsonify(f"Book #{book.id} successfully updated"))
  ```

  **Updated tests:**
  ```py
  def test_update_book_without_title(client, two_saved_books):
      # Arrange
      book_id = 1
      data = {
          "description": "New description"
      }

      # Act
      response = client.put(f"/books/{book_id}", json=data)
      response_body = response.get_json()

      # Assert
      assert response.status_code == 400
      assert response_body == {"message": "title is required"}

  def test_update_book_without_description(client, two_saved_books):
      # Arrange
      book_id = 1
      data = {
          "title": "Updated Book"
      }

      # Act
      response = client.put(f"/books/{book_id}", json=data)
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
    except:
        abort(make_response({"message": f"{model_class.__name__.lower()} {model_id} invalid"}, 400))

    model = model_class.query.get(model_id)

    if not model:
        abort(make_response({"message": f"{model_class.__name__.lower()} {model_id} not found"}, 404))

    return model
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