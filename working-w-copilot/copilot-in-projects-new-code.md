#  Copilot in Projects Pt. 1 - Writing New Code

## Goals

We know how to generate code, start a chat, and work with Copilot's tools in VS Code, so now let's see what it looks like to apply what we've learned in a project. We'll mosey back to the [hello-books example project](https://github.com/AdaGold/hello-books-api) to see how we can use Copilot to increase our productivity.

Our goals for this lesson are to show how we can use Copilot to:
- quickly write new code that follows a consistent format
- help us with generating test cases

## Writing new code with Copilot

We won't be following the same path as our previous visit with `hello-books`. To keep things shorter and focus on areas where we can benefit from Copilot, we will check out a couple branches and use the code at that state as an example to work from.

For now, we're going to start at the branch [`03a-models-setup`](https://github.com/AdaGold/hello-books-api/tree/03a-models-setup) so that the database connection is set up for us in our `__init__.py` and we have a single model class `Book` already existing.

Our plan in this section of the lesson is to:
- Add the convenience methods `from_dict` and `to_dict` to the existing `Book` model
- Write tests for our new functions
- Create the `Author`, `Genre`, and `BookGenre` model classes.

### Adding convenience functions to `Book`

Opening the `Book` class, we have the bare bones of a model defining 3 properties: `id`, `title`, and `description`. Our first step will be to add our `to_dict` function. To make GET requests easier, we know that we want a function that takes a `Book` model and converts it to a dictionary for us, so let's write a comment to that effect. Add the following comment to your `book.py` file then press enter to see what Copilot suggests:

```py
# An instance function that returns a  
# dictionary which represents the book model
```

In our case, Copilot immediately suggests a function signature and body that matches our needs for the moment:

![VS Code window showing the comment above along with a function suggested by copilot](assets/copilot-in-projects/book-to-dict-suggestion.png)  
*Fig. Our function description comment above with a suggestion for the `to_dict` function from Copilot in grey below*

We could use a comment to try generating our `from_dict` class method, but instead we'll use the inline chat to prompt Copilot. Let's use `⌘I` (`CMD + i`) to open an inline chat and enter the following:

> Please write a class function that takes in a dictionary and returns a new Book instance created from the input dictionary's contents

When we submit the prompt, we are likely to receive a response similar to the following:

![VS Code showing the Copilot inline chat with the prompt above entered and a function suggestion below](assets/copilot-in-projects/book-from-dict-initial-suggestion.png)  
*Fig. Our prompt entered in the Copilot inline chat with a suggestion for the `from_dict` function from Copilot in a temporary window below*

The suggestion is succinct and will create a new class instance for us from a dictionary, but is this what we actually want? What happens if the dictionary is missing `title` or `description` keys? Does this take advantage of the `autoincrement` feature of SQLAlchemy that we're using for the `id`?

### !callout-info

## Reviewing the `**` operator

The Python `**` operator unpacks the contents of a `dict` so that its keys and values can be used as named arguments to a function. In this case, Copilot has suggested a function that takes in a dictionary and uses the `**` operator to unpack the dictionary's contents into arguments to the `Book` class's `__init__` method, which will be called through the `cls` reference.

### !end-callout

No, this isn't what we want! We have no error handling for missing properties, and this one-liner will set a Book's `id` to whatever value is in the `id` key of the dictionary if any—the `id` property's `autoincrement` flag will be ignored.

We need a more robust function, so let's update our prompt. There are 2 significant features missing: error handling and utilizing `autoincrement`.

<br />

<details>
  <summary>
    Take a moment to try adding to the original prompt yourself, then expand this section to see our updated prompt and what Copilot suggested.
  </summary>

**Updated prompt:**
> Please write a class function that takes in a dictionary and returns a new Book instance created from the input dictionary's contents. The function should include error handling for missing values and should take advantage of the autoincrement feature.


**Suggestion from Copilot:** 
```py
@classmethod
def from_dict(cls, data):
    if 'title' not in data or 'description' not in data:
        raise ValueError("Missing required fields: title or description")
    return cls(title=data['title'], description=data['description'])
```

Our final prompt doesn't change our initial sentence; we were able to add one new sentence that encapsulated our requirements, and all of a sudden we have a suggestion that meets our current needs!

The method might not look exactly as we would have written it, but if it bothers us or doesn't follow our team or project's conventions, we can always update the code after accepting the suggestion. For example, we may need to modify the generated error string. Or perhaps seeing this code reminds us that we could watch for a `KeyError` rather than checking for individual keys in the dictionary. Copilot is not meant to replace our critical thinking, but it can help us get started on the right path!
</details>

### Testing the Book class

These functions aren't very long, but it's still a good idea to test them as a baseline for any future changes to this class or related code. We'll use Copilot to help us get started on brainstorming unit tests from the inline chat.

In our `book.py` file, highlight the whole text, bring up the inline Copilot chat with `⌘I` (`CMD + i`), then type in the shortcut `/tests`.

![VS Code showing the full book.py file contents highlighted with the Copilot inline chat bar showing and "/tests" typed in](assets/copilot-in-projects/book-slash-tests-start.png)  
*Fig. Selected text in* `book.py` *with the Copilot inline chat up to enter "/tests"*

Once we hit `Enter`, Copilot will add a new pane in the VS Code window with our copilot chat at the top and a temporary file with unit tests that we can review.

![VS Code showing the book.py class and a temporary file where Copilot is previewing unit tests](assets/copilot-in-projects/book-slash-tests-suggestion.png)  
*Fig. Copilot's UI to preview tests for the* `book.py` *file*

If we feel like the tests presented are a good starting place, we can click "Accept" in the copilot chat to dismiss the chat window, then use `⌘S` (`CMD + S`) to save the new test file. We should see a pop up that allows us to choose the folder where we want to save the test file.

We must carefully review the tests that Copilot generates for things like missing cases or tricky edge cases. We may get lucky and have all of our bases covered, but we'll often want to add or update the tests slightly. In our case, Copilot came up with 3 tests that nearly have us covered:

```py
import pytest
from .book import Book

def test_to_dict():
    book = Book(id=1, title="Test Title", description="Test Description")
    expected = {
        'id': 1,
        'title': "Test Title",
        'description': "Test Description"
    }
    assert book.to_dict() == expected

def test_from_dict_success():
    data = {
        'title': "Test Title",
        'description': "Test Description"
    }
    book = Book.from_dict(data)
    assert book.title == "Test Title"
    assert book.description == "Test Description"

def test_from_dict_missing_title():
    data = {
        'description': "Test Description"
    }
    with pytest.raises(ValueError, match="Missing required fields: title or description"):
        Book.from_dict(data)
```

### !callout-warning

## Creating `Book` instances in tests
In our first test, `test_book_to_dict`, the `Book` class is instantiated from a dictionary with an explicit `id`, even though we discussed avoiding setting `id`s ourselves when creating the `from_dict` method. 

<br />

In this specific case, we're testing the `to_dict` method, which is isolated from our routes code or other database access. However we arrange the `Book` for testing this particular method is fine, but there are many instances where we need to use our fixtures to create Book instances that are tracked by SQLAlchemy and exist in our database for routes to retrieve. 

<br />

When we use Copilot to create tests, we should always evaluate if the data is being set up correctly in the `Arrange` step for all of the actions we will be taking.

### !end-callout

We have tests to ensure that for a given book model, the dictionary that `to_dict` creates contains the right data, and that a book created by `from_dict` stores the title and description from the input dictionary into the right properties. Our last test ensures that if the `"title"` key is missing from the input dictionary, we raise an error. But what if the `"description"` key is missing instead?

To make our test suite as complete as possible, we can add one more test `test_book_from_dict_missing_description` to make sure that we have this edge case covered. We can write the additional test by hand, but we can also try highlighting the last test and asking Copilot to do that work for us!

<br />

<details>
  <summary>
    Try it out yourself, then expand this section to see how we asked Copilot to handle the test updates.
  </summary>

  **Prompt:**
  > Please add a new test for the from_dict function that tests the scenario where the description is missing but the title is present

  **Newly added test:**
  ```py
  def test_from_dict_missing_description():
    data = {
        'title': "Test Title"
    }
    with pytest.raises(ValueError, match="Missing required fields: title or description"):
        Book.from_dict(data)
  ```
</details>

### Creating `Author`, `Genre`, and `BookGenre` models

We can ask Copilot to help us write something even if we don't have a template or example, but Copilot tends to produce more relevant results if we have samples to show. Here, we have a `Book` model that we can use as a pattern when asking Copilot to help us create the `Author`, `Genre`, and `BookGenre` models.

Before we ask Copilot for help, the first thing we need to do is gather the requirements for our new classes. In our previous work with these models, we gave them the following properties:

**Author**
- autoincrementing `id`
- string `name`
- a property to access `Books` associated with a particular `Author` instance

**Genre**
- autoincrementing `id`
- string `name`

**BookGenre**
- foreign key connection to the `Book` model
- foreign key connection to the `Genre` model

Now that we've reminded ourselves of what the classes need to do, we can use these details to craft a prompt for Copilot. This time, let's use `⌃⌘I` (`CTRL + CMD + i`) to open up the Copilot chat pane. We can type directly in the chat box, but it can be helpful to write up prompts in a text editor first, especially if they span multiple lines.

<br />

<details>
  <summary>
    Before continuing, pause for a moment and try to write a prompt that uses our class requirements to describe what we want from Copilot. When you're done, expand this section to see the prompt we used.
  </summary>

  **Our Prompt:**
  > I would like help creating three more model classes that follow the same pattern as the Book class.
  >
  > The first new model should be named Author and have 3 properties: an auto incrementing `id`, a string `name`, and a relationship to the Book model that back populates a property named `author`.
  >
  > The second new model should be named Genre and have 2 properties: an auto incrementing `id` and a string `name`.
  >
  > The last new model should be named BookGenre and act as a join table between the Book and Genre models.
</details>

Being an AI tool, we will likely see slightly different responses, even with the same prompt. When we submit our prompt, Copilot will create suggestions for us in the chat pane, and we are given some controls of what to do with that code. In the screenshot below, the controls from left to right are:
- **Apply In Editor** - replaces the contents of the current file with the generated code
- **Insert at Cursor** - adds the code directly to an open file where the cursor currently is
- **Copy** - adds all the code to the clipboard to paste where you'd like
- **More Actions** - shows a dropdown with the options below
  - **Insert into Terminal**
  - **Insert into New File**

![A section of the Copilot chat pane in VS Code showing the controls for code suggestions](assets/copilot-in-projects/copilot-chat-pane-code-controls.png)  
*Fig. Copilot Chat's controls for code suggestions*

Before we make any decisions about where we want to put the code, let's examine what Copilot suggested for us.

> Sure, here are the three new model classes following the same pattern as the Book class:
> 
> ```py
> from sqlalchemy.orm import Mapped, mapped_column, relationship
> from ..db import db
> 
> class Author(db.Model):
>     id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
>     name: Mapped[str]
>     books: Mapped[list["Book"]] = relationship("Book", back_populates="author")
> 
>     def to_dict(self):
>         return {
>             'id': self.id,
>             'name': self.name,
>             'books': [book.to_dict() for book in self.books]
>         }
> 
>     @classmethod
>     def from_dict(cls, data):
>         if 'name' not in data:
>             raise ValueError("Missing required field: name")
>         return cls(name=data['name'])
> 
> 
> class Genre(db.Model):
>     id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
>     name: Mapped[str]
> 
>     def to_dict(self):
>         return {
>             'id': self.id,
>             'name': self.name
>         }
> 
>     @classmethod
>     def from_dict(cls, data):
>         if 'name' not in data:
>             raise ValueError("Missing required field: name")
>         return cls(name=data['name'])
> 
> 
> class BookGenre(db.Model):
>     book_id: Mapped[int] = mapped_column(db.ForeignKey('book.id'), primary_key=True)
>     genre_id: Mapped[int] = mapped_column(db.ForeignKey('genre.id'), primary_key=True)
> 
>     book: Mapped["Book"] = relationship("Book", back_populates="genres")
>     genre: Mapped["Genre"] = relationship("Genre", back_populates="books")
> 
>     def to_dict(self):
>         return {
>             'book_id': self.book_id,
>             'genre_id': self.genre_id
>         }
> 
>     @classmethod
>     def from_dict(cls, data):
>         if 'book_id' not in data or 'genre_id' not in data:
>             raise ValueError("Missing required fields: book_id or genre_id")
>         return cls(book_id=data['book_id'], genre_id=data['genre_id'])
> ```
> Please note that you need to add a foreign key field in the `Book` model to reference the `Author` model. Also, the `BookGenre` model is a many-to-many relationship table between `Book` and `Genre`. It has two primary keys, `book_id` and `genre_id`, which are foreign keys referencing `Book` and `Genre` respectively.

Overall, this is a solid start! The classes all meet the property requirements we outlined, the `Author` and `Genre` classes have the `to_dict` and `from_dict` convenience functions defined, and Copilot even gave us a heads up about the changes we'll need to make to the `Book` class for everything to work.

We could implement those changes ourselves, but we can get help from Copilot with this task with a prompt like:

> Please update the Book model class to add foreign keys and relationship attributes to create the one to many relationship between Authors and Books and to create the many to many relationship between books and genres.

**Copilot's response:**
> Sure, here is the updated `Book` model class with the necessary foreign keys and relationship attributes:
> ```py
> from sqlalchemy.orm import Mapped, mapped_column, relationship
> from ..db import db
> 
> class Book(db.Model):
>     id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
>     title: Mapped[str]
>     description: Mapped[str]
>     author_id: Mapped[int] = mapped_column(db.ForeignKey('author.id'))
>     author: Mapped["Author"] = relationship("Author", back_populates="books")
>     genres: Mapped[list["BookGenre"]] = relationship("BookGenre", back_populates="book")
> 
>     def to_dict(self):
>         return {
>             'id': self.id,
>             'title': self.title,
>             'description': self.description,
>             'author': self.author.to_dict() if self.author else None,
>             'genres': [genre.to_dict() for genre in self.genres]
>         }    
> 
>     @classmethod
>     def from_dict(cls, data):
>         if 'title' not in data or 'description' not in data:
>             raise ValueError("Missing required fields: title or description")
>         return cls(title=data['title'], description=data['description'])
> ```
> This code adds the `author_id` foreign key and the relationships to the `Author` and `BookGenre` models, establishing the one-to-many relationship between `Author` and `Book` and the many-to-many relationship between `Book` and `Genre`.

The updated code that Copilot displays is looking pretty good, the `Book` suggestion has all the new properties we needed to create our one-to-many `Author` relationship and many-to-many `Genre` relationship.

We can accept the code as is, but there is still some room for improvement depending on our needs. In the `to_dict` functions for `Book` and `Author`, we may want to limit what information is sent forward about our newly added relationships. Right now, `to_dict` is called on the new relationship objects so all of their information including ids and information about _their_ relationships gets packaged up in the response. 
- For the `Book` class, it would be nice to include only the author's name and the names of the genres associated with the book in the return dictionary.
- For `Author`, it could be handy if the return dictionary included only the `title`s of the `Book` instances that the Author is associated with. 

### !callout-warning

## Consider Including List Values Carefully

In this case, we're expecting that a particular book will have only a few genres related to it, and any given author will have written only so many books. If we were working with data where we expected a large number of related items, we might not want to include them in the basic `to_dict` function, which primarily serves the purpose of making it easier to return data for a GET request.

<br />

Instead, we might wait to return those lists until we're specifically asked for them from a related endpoint. In the GET response, we might include a count of the number of related items, and then the client could make a separate request to get the full list of related items if they wanted to.

### !end-callout

We could make the changes manually after moving the code to a file, but we can also ask Copilot to make these updates.

<br />

<details>
  <summary>
    Feel free to try out asking Copilot to help with the changes to the <code>Book</code> and <code>Author</code> <code>to_dict</code> functions. When you're done, expand this section to see the prompt we used and what Copilot suggested.
  </summary>

  **Updated Prompt:**
  > Please update the Book to_dict function to include only the name of the author and a list of the names of the genres associated in the return dictionary, if those pieces of data exist. Please update the Author to_dict function to include only the title of each item in the list books.

  **Copilot's Suggested Code:**
  > ```py
  > from sqlalchemy.orm import Mapped, mapped_column, relationship
  > from ..db import db
  > 
  > class Book(db.Model):
  >     id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
  >     title: Mapped[str]
  >     description: Mapped[str]
  >     author_id: Mapped[int] = mapped_column(db.ForeignKey('author.id'))
  >     author: Mapped["Author"] = relationship("Author", back_populates="books")
  >     genres: Mapped[list["BookGenre"]] = relationship("BookGenre", back_populates="book")
  > 
  >     def to_dict(self):
  >         return {
  >             'id': self.id,
  >             'title': self.title,
  >             'description': self.description,
  >             'author': self.author.name if self.author else None,
  >             'genres': [genre.genre.name for genre in self.genres] if self.genres else []
  >         }    
  >     
  >     @classmethod
  >     def from_dict(cls, data):
  >         if 'title' not in data or 'description' not in data:
  >             raise ValueError("Missing required fields: title or description")
  >         return cls(title=data['title'], description=data['description'])
  > ```
  > ```py
  > from sqlalchemy.orm import Mapped, mapped_column, relationship
  > from ..db import db
  > 
  > class Author(db.Model):
  >     id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
  >     name: Mapped[str]
  >     books: Mapped[list["Book"]] = relationship("Book", back_populates="author")
  > 
  >     def to_dict(self):
  >         return {
  >             'id': self.id,
  >             'name': self.name,
  >             'books': [book.title for book in self.books]
  >         }
  > 
  >     @classmethod
  >     def from_dict(cls, data):
  >         if 'name' not in data:
  >             raise ValueError("Missing required field: name")
  >         return cls(name=data['name'])
  > ```
</details>

There may yet be further things to consider, one we can see is that the `Book` model's `from_dict` method could be updated to handle an `author_id` or `genre_id` value. However, once we are happy with the presented code we can copy our updated `Book`, and new `Author`, `Genre`, and `BookGenre` classes from the Copilot chat to their own files. From there we should wrap up our changes by updating the tests for our Book model with our new scenarios, and create tests for our new models—all on our own or with Copilot's assistance!

## Summary

Copilot can help make many code tasks move faster, as long as we use it with caution. There are many ways to prompt Copilot to suggest code, but no matter how we access Copilot in the UI, we need to make sure that we give Copilot enough context to meaningfully help us. That might mean making sure we have files open that Copilot can use as examples, or writing detailed prompts that cover the requirements of the code we want to generate. Copilot can help us with brainstorming test cases, but there is no guarantee that Copilot will suggest a complete suite that covers all of our important edge cases, so we need to review what's presented critically and expand the suite if necessary.

## Check for Understanding

<!-- prettier-ignore-start -->
### !challenge
* type: checkbox
* id: 9100b9a2-b806-4b0c-94d2-5f1af5acf54b
* title: Copilot in Projects Pt. 1 - Writing New Code

##### !question
What are some points to look out for when using the `/tests` shortcut.
##### !end-question

##### !options
a| Do the tests cover all of the edge cases we need?
b| Are the right imports included when referencing other classes or files?
c| Was the test file generated in the source folder instead of the test folder?
d| Do the test assertions assume the correct kind of data?
##### !end-options

##### !answer
a|
b|
c|
d|
##### !end-answer

### !end-challenge
<!-- prettier-ignore-end -->
