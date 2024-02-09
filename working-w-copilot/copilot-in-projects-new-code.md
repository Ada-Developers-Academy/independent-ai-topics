#  Copilot in Projects Pt. 1 - Writing New Code

## Goals

We know how to generate code, start a chat, and work with Copilot's tools in VS Code, but what does it look like to apply what we've learned in a project? We'll mosey back to the [hello-books example project](https://github.com/AdaGold/hello-books-api) from Unit 2 to see how we can use Copilot to increase our productivity.

Our goals for this lesson are to show how we can use Copilot to:
- quickly write new code that follows a consistent format
- help us with generating test cases 

## Writing new code with Copilot

We will not be following the same path as Unit 2 did with `hello-books`. To keep things shorter and focus on areas where we can benefit from Copilot we will check out a couple branches and use the code at that state as an example to work from.

For now, we're going to start at the branch [`03a-models-setup`](https://github.com/AdaGold/hello-books-api/tree/03a-models-setup) so that the database connection is set up for us in our `__init__.py` and we have a single model class `Book` already existing. 

Our plans in this section of the lesson are to:
- Add the convenience methods `from_dict` & `to_dict` to the existing `Book` model 
- Write tests for our new functions
- Create the `Author`, `Genre`, and `BookGenre` model classes.

### Adding convenience functions to `Book`

If we open up the `Book` class, we should have the bare bones of a model defining 3 properties: `id`, `title`, and `description`. Our first step will be to add our `to_dict` function. To make GET requests easier, we know that we want a function that takes a `Book` model and converts it to a dictionary for us, so let's write a comment to that effect. Add the following comment to your `Book.py` file then press enter to see what Copilot suggests:

```py
# An instance function that returns a  
# dictionary which represents the book model
```

In our case, Copilot immediately suggested a function signature and body that matched our needs for the moment:

![VS Code window showing the comment above along with a function suggested by copilot](assets/copilot-in-projects/book-to-dict-suggestion.png)  
*Fig. Our function description comment above with a suggestion for the `to_dict` function from Copilot in grey below*

We could use a comment to try generating our `from_dict` class method, but instead we'll use the inline chat to prompt Copilot. Let's use `CMD + i` to open an inline chat and enter the following:

> Please write a class function that takes in a dictionary and returns a new Book instance created from the input dictionary's contents

When we submit the prompt, we may see a response like:

![VS Code showing the Copilot inline chat with the prompt above entered and a function suggestion below](assets/copilot-in-projects/book-from-dict-initial-suggestion.png)  
*Fig. Our prompt entered in the Copilot inline chat with a suggestion for the `from_dict` function from Copilot in a temporary window below*

The suggestion is succinct and will create a new class instance for us from a dictionary, but is this what we actually want? What happens if the dictionary is missing `title` or `description` keys? Does this take advantage of the `autoincrement` feature of SQLAlchemy that we're using for the `id`?

No, this isn't what we want! We have no error handling for missing properties, and this one-liner will set a Book's `id` to whatever is in the `id` key of the dictionary if it exists – the `id` property's `autoincrement` flag will be ignored. 

We need a more robust function, so let's update our prompt. There are 2 significant features missing: error handling and utilizing `autoincrement`. 

<details>
  <summary>
    Take a moment to try adding to the original prompt yourself, then expand this section to see our updated prompt and what Copilot suggested.
  </summary>

**Updated prompt:**   
> Please write a class function that takes in a dictionary and returns a new Book instance created from the input dictionary's contents. The function should include error handling for missing values and should take advantage of the autoincrement feature.


**Suggestion from Copilot: ** 
```py
@classmethod
def from_dict(cls, data):
    if 'title' not in data or 'description' not in data:
        raise ValueError("Missing required fields: title or description")
    return cls(title=data['title'], description=data['description'])
```

Our final prompt doesn't change our initial sentence; we were able to add one new sentence that encapsulated our requirements, and all of a sudden we have a suggestion that meets our current needs! The method may not look exactly as we might have written it out, but if it bothers us or doesn't follow our team or project's conventions, we can always update the code after accepting the suggestion.  
</details>

### Testing the Book class

These functions aren't very long, but it's still a good idea to test them in case of any future changes to this class or related code. We'll use Copilot to help us get started on brainstorming unit tests from the inline chat.

In our `Book.py` file, highlight the whole text, bring up the inline Copilot chat with `CMD + i`, then type in the shortcut `/tests`.

![VS Code showing the full Book.py file contents highlighted with the Copilot inline chat bar showing and "/tests" typed in](assets/copilot-in-projects/book-slash-tests-start.png)  
*Fig. Selected text in Book.py with the Copilot inline chat up to enter "/tests"*

Once we hit `Enter` Copilot will create a temporary section on the VS Code window with unit tests that we can review. 

![VS Code showing the Book.py class and a temporary window where Copilot is previewing unit tests](assets/copilot-in-projects/book-slash-tests-suggestion.png)
*Fig. Copilot's UI to preview tests for the Book.py file*

If we feel like the tests presented are a good starting place, we can use the "Create" button to make a new file with those tests pre-populated. 

### !callout-warning

## Test File Location

The "Create" button will generate the new test file in the current folder, in our case, in the same folder as our `Book` model. This isn't really what we want, but we can click the arrow next to the "Create" button to see a drop down with a "Create As" option. This will let us change the default name of the file and choose where it gets saved so that we can keep all of our test files together, separate from our source code. If you don't use "Create As", you can drag and drop test files from where they were created to the test folder at any later time.

![Close up of the Copilot window for previewing tests showing the drop down of options off of the "Create" button](assets/copilot-in-projects/book-slash-tests-create-as.png)  
*Fig. "Create As" in the drop down next to the "Create" button*

### !end-callout

We want to carefully review the tests that Copilot generates for things like missing cases or tricky edge cases. We may get lucky and have all of our bases covered, but we'll often want to add or update the tests slightly. In our case, Copilot came up with 3 tests that nearly have us covered:

```py
def test_book_to_dict():
    book_data = {
        'id': 1,
        'title': 'The Hobbit',
        'description': 'A book about a hobbit'
    }
    book = Book(**book_data)
    book_dict = book.to_dict()

    assert book_dict == book_data

def test_book_from_dict():
    book_data = {
        'title': 'The Hobbit',
        'description': 'A book about a hobbit'
    }
    book = Book.from_dict(book_data)

    assert book.title == 'The Hobbit'
    assert book.description == 'A book about a hobbit'

def test_book_from_dict_missing_fields():
    book_data = {
        'title': 'The Hobbit'
    }

    with pytest.raises(ValueError):
        Book.from_dict(book_data)
```

We have tests that ensure that for a given book model, the dictionary that `to_dict` creates contains the right data, and that a book created by `from_dict` stores the title and description from the input dictionary into the right properties. Our last test ensures that if the 'description' key is missing from the input dictionary that we raise an error, but what if the title is missing instead? 

To make our test suite as complete as possible, we can update the name of our last test to `test_book_from_dict_missing_description` and add one more test `test_book_from_dict_missing_title` to make sure we have that edge case covered. We can do this by hand, but feel free to try out highlighting the last test and asking Copilot to do that work for us!

<details>
  <summary>
    Try it out yourself, then expand this section to see how we asked Copilot to handle the test updates.
  </summary>

  **Updated prompt:** 
  > Please rename this test to reflect that it only tests what happens if the description field is missing. Next, add a new test for the from_dict function that tests the scenario where the title is missing but the description is present

  **Updated test and newly added test:**  
  ```py
  def test_book_from_dict_missing_description():
      book_data = {
          'title': 'The Hobbit'
      }

      with pytest.raises(ValueError):
          Book.from_dict(book_data)

  def test_book_from_dict_missing_title():
      book_data = {
          'description': 'A book about a hobbit'
      }

      with pytest.raises(ValueError):
          Book.from_dict(book_data)
  ```
</details>

### Creating `Author`, `Genre`, and `BookGenre` models

We can ask Copilot to help us write something even if we don;t have a template or example, but it's easier for Copilot to reason about what we want if we have samples to show. Here, we have a `Book` model that we can use as a pattern when asking Copilot to help us create the `Author`, `Genre`, and `BookGenre` models. 

Before we ask Copilot for help, the first thing we need to do is gather the requirements for our new classes. If we recall back to the Unit 2 lessons, we want our models to have the following properties:

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

Now that we're reminded ourselves of what the classes need to do, we can use these details to craft a prompt for Copilot. This time, let's use `CTRL + CMD + i` to open up the Copilot chat pane. We can type directly in the chat box, but it can be helpful to write up prompts in a text editor first, especially if they span multiple lines. 

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
- Copy - add all the code to the clipboard to paste where you'd like
- Insert at Cursor - will add the code directly to an open file where your cursor currently is
- More Actions - When clicked will show a dropdown with the options below
  - Insert into New File
  - Insert into Terminal

![A section of the Copilot chat pane in VS Code showing the controls for code suggestions](assets/copilot-in-projects/copilot-chat-pane-code-controls.png)  
*Fig. Copilot Chat's controls for code suggestions*

Before we make any decisions about where we want to put the code, let's examine what Copilot suggested for us.

> Sure, here are the three new model classes following the same pattern as the Book class:
> 
> ```py
> class Author(db.Model):
>     id = db.Column(db.Integer, primary_key=True, autoincrement=True)
>     name = db.Column(db.String)
>     books = db.relationship('Book', backref='author', lazy=True)
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
> class Genre(db.Model):
>     id = db.Column(db.Integer, primary_key=True, autoincrement=True)
>     name = db.Column(db.String)
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
> class BookGenre(db.Model):
>     book_id = db.Column(db.Integer, db.ForeignKey('book.id'), primary_key=True)
>     genre_id = db.Column(db.Integer, db.ForeignKey('genre.id'), primary_key=True)
>     book = db.relationship('Book', backref=db.backref('book_genres', cascade='all, delete'))
>     genre = db.relationship('Genre', backref=db.backref('book_genres', cascade='all, delete'))
> ```
> Please note that you need to add a foreign key field in the `Book` model to reference the `Author` model. Also, the `BookGenre` model is a many-to-many relationship table between `Book` and `Genre`. It has two primary keys, `book_id` and `genre_id`, which are foreign keys referencing `Book` and `Genre` respectively.

Overall, this is a solid start! The classes all meet the property requirements we outlined, the `Author` and `Genre` classes have the `to_dict` and `from_dict` convenience functions defined, and Copilot even gave us a heads up about the changes we'll need to make to the `Book` class for everything to work.

However, the code isn't perfect – the Author and BookGenre classes are using a deprecated keyword `backref` whenever creating a relationship with another model. We want to replace the uses of `backref` with the `back_populates` syntax, and we also still need to update our `Book` class to work with these new classes. 

We can get help from Copilot with both of these tasks at the same time with a prompt like:  

> Please show all of the model classes including Book with updates to use back_populates instead of backref

**Copilot's response:**
> Sure, here are all the model classes including `Book` with the updates to use `back_populates` instead of `backref`:
> ```py
> from app import db
>  
> class Book(db.Model):
>     id = db.Column(db.Integer, primary_key=True, autoincrement=True)
>     title = db.Column(db.String)
>     description = db.Column(db.String)
>     author_id = db.Column(db.Integer, db.ForeignKey('author.id'))
>     author = db.relationship('Author', back_populates='books')
>     book_genres = db.relationship('BookGenre', back_populates='book')
>  
>     def to_dict(self):
>         return {
>             'id': self.id,
>             'title': self.title,
>             'description': self.description
>         }
>  
>     @classmethod
>     def from_dict(cls, data):
>         if 'title' not in data or 'description' not in data:
>             raise ValueError("Missing required fields: title or description")
>         return cls(title=data['title'], description=data['description'])
>  
> class Author(db.Model):
>     id = db.Column(db.Integer, primary_key=True, autoincrement=True)
>     name = db.Column(db.String)
>     books = db.relationship('Book', back_populates='author')
>  
>     def to_dict(self):
>         return {
>             'id': self.id,
>             'name': self.name,
>         }
>  
>     @classmethod
>     def from_dict(cls, data):
>         if 'name' not in data:
>             raise ValueError("Missing required field: name")
>         return cls(name=data['name'])
>  
> class Genre(db.Model):
>     id = db.Column(db.Integer, primary_key=True, autoincrement=True)
>     name = db.Column(db.String)
>     book_genres = db.relationship('BookGenre', back_populates='genre')
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
> class BookGenre(db.Model):
>     book_id = db.Column(db.Integer, db.ForeignKey('book.id'), primary_key=True)
>     genre_id = db.Column(db.Integer, db.ForeignKey('genre.id'), primary_key=True)
>     book = db.relationship('Book', back_populates='book_genres')
>     genre = db.relationship('Genre', back_populates='book_genres')
> ```
> Please note that the back_populates option is used in both sides of the relationship and must refer to the other side's attribute name that defines the relationship.

The updated code that Copilot displays is looking much better! The deprecated `backref` usage has been replaced and the `Book` suggestion has all the new properties we needed to create our one-to-many `Author` relationship and many-to-many `Genre` relationship. 

We can accept the code as-is, but there is still some room for improvement depending on our needs. There are properties like `name` that we might want to mark as required on some of our models. In the `to_dict` functions for `Book` and `Author`, we have some properties that aren't being included. 
- For the `Book` class, it would be nice to also include the author's name and genres in the return dictionary.
- For `Author`, it could be handy if the return dictionary included a list of the `title`s of the `Book` instances that the Author is associated with. 

We could make the changes manually after moving the code to a file, but we can also ask Copilot to make these updates.

<details>
  <summary>
    We will leave marking properties as required as a personal exercise, but feel free to try out asking Copilot to help with the changes to the <code>Book</code> and <code>Author</code> <code>to_dict</code> functions. When you're done, expand this section to see the prompt we used and what Copilot suggested.
  </summary>

  **Updated Prompt:**
  > Please update the Book to_dict function to include the name of the author and a list of the names of the genres associated in the return dictionary, only if those pieces of data exist. Please update the Author to_dict function to include the title of each item in the list books.

  **Copilot's Suggested Code:**
  ```py
  class Book(db.Model):
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    title = db.Column(db.String)
    description = db.Column(db.String)
    author_id = db.Column(db.Integer, db.ForeignKey('author.id'))
    author = db.relationship('Author', back_populates='books')
    book_genres = db.relationship('BookGenre', back_populates='book')

    def to_dict(self):
        data = {
            'id': self.id,
            'title': self.title,
            'description': self.description
        }
        if self.author:
            data['author'] = self.author.name
        if self.book_genres:
            data['genres'] = [bg.genre.name for bg in self.book_genres]
        return data

    @classmethod
    def from_dict(cls, data):
        if 'title' not in data or 'description' not in data:
            raise ValueError("Missing required fields: title or description")
        return cls(title=data['title'], description=data['description'])

  class Author(db.Model):
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    name = db.Column(db.String)
    books = db.relationship('Book', back_populates='author')

    def to_dict(self):
        return {
            'id': self.id,
            'name': self.name,
            'books': [{'id': book.id, 'title': book.title} for book in self.books]
        }

    @classmethod
    def from_dict(cls, data):
        if 'name' not in data:
            raise ValueError("Missing required field: name")
        return cls(name=data['name'])
  ```
</details>

Once we are happy with the presented code we can copy our updated `Book` and new `Author`, `Genre`, and `BookGenre` classes from the Copilot chat to their own files. From there we should wrap up our changes by updating the tests for our Book model with our new scenarios, and create tests for our new models – all on our own or with Copilot's assistance!

## Summary

Copilot can help make many code tasks move faster, as long as we use it with caution. There are many ways to prompt Copilot to suggest code, but however we are accessing Copilot in the UI, we need to make sure that we are giving Copilot enough context to meaningfully help us. That might mean making sure we have files open that Copilot can use as examples, or writing detailed prompts that cover the requirements of the code we want to generate. Copilot can help us with brainstorming test cases, but there is no guarantee that Copilot will suggest a complete suite that covers all of our important edge cases, so we need to review what's presented critically and expand the suite if necessary.

## Check for Understanding

<!-- Question 1 -->
<!-- prettier-ignore-start -->
### !challenge
* type: checkbox
* id: 9100b9a2-b806-4b0c-94d2-5f1af5acf54b
* title: Using `/tests`

##### !question
Select the options below that we need to look out for when using the `/tests` shortcut.
##### !end-question

##### !options
* The tests being placed in the source file when accepted
* If Copilot brought in the right imports when referencing other classes or files
* A test file being generated in the source folder instead of test folder
* Whether the test assertions are assuming the correct kind of data
##### !end-options

##### !answer
* If Copilot brought in the right imports when referencing other classes or files
* A test file being generated in the source folder instead of test folder
* Whether the test assertions are assuming the correct kind of data
##### !end-answer

### !end-challenge
<!-- prettier-ignore-end --> 
