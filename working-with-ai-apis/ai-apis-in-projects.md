# Using the Google Gemini API

## Goals

Now that we know more about requests and responses to the `/gemini-2.5-flash:generateContent` endpoint, let's learn how to use it within our programs. To do so, we will work with a partially finished API that creates video game NPCs (Non Playable Characters). Within that program, we will install the [`google-genai`](https://pypi.org/project/google-genai/) Python package and then leverage the `gemini-2.5-flash` model's `generateContent` function to generate dialogue for characters we create.

Our goals for this lesson are to:
- Integrate the `google-genai` library into a Python project.
- Use the `/gemini-2.5-flash:generateContent` text completion endpoint to generate content for our project. 

## Set Up the NPC Generator

This tutorial will walk you through how to integrate the Gemini API into your projects with the companion repo [NPC Generator](https://github.com/AdaGold/NPC-Generator). We'll start with our usual setup.

1. Fork the repo to your GitHub account and then clone it to your machine.
2. Create and run a virtual environment for the project.
3. Use `pip` to install the `requirements.txt`.

## Explore the NPC Generator repo

Take some time to explore the NPC Generator repo in its current state. For this project, we will be finishing an API for a role-playing style game. Our API creates NPCs (Non Playable Characters) that your player's character will interact with. 

We currently have two models, "Character" and "Greeting". "Character" holds some basic information about a character and has a one to many relationship with "Greeting", which represents a stock phrase that a particular character may say when interacting with the player. Each NPC can have multiple greetings but each greeting can only be connected to one character. We will use the Gemini API to generate those greetings which we will then store in our database.

Feel free to expand the sections below to see our model code.

<br />

<details>
  <summary><code>Character</code> model</summary>

```python
from sqlalchemy.orm import Mapped, mapped_column, relationship
from ..db import db

class Character(db.Model):
    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    name: Mapped[str]
    personality: Mapped[str]
    occupation: Mapped[str]
    age: Mapped[int]
    greetings: Mapped[list["Greeting"]] = relationship(back_populates="character")
    
    def to_dict(self):
        return {
            "id" : self.id,
            "name" : self.name,
            "personality" : self.personality,
            "occupation" : self.occupation,
            "age" : self.age
        }
    
    @classmethod
    def from_dict(cls, data_dict):
        new_character = cls(
            name = data_dict["name"],
            personality = data_dict["personality"],
            occupation = data_dict["occupation"],
            age = data_dict["age"]
        )

        return new_character
```
</details>
</br>
<br />

<details>
  <summary><code>Greeting</code> model</summary>

```python
from sqlalchemy.orm import Mapped, mapped_column, relationship
from sqlalchemy import ForeignKey
from ..db import db

class Greeting(db.Model):
    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    greeting_text: Mapped[str]
    character_id: Mapped[int] = mapped_column(ForeignKey("character.id")) 
    character: Mapped["Character"] = relationship(back_populates="greetings")

    def to_dict(self):
        return {
            "id" : self.id,
            "greeting" : self.greeting_text,
        }
```
</details>
</br>

Now let's take a look at the routes we currently have. If you examine the code below, we currently have our `validate_model` helper method as well as a POST endpoint to add a character and a GET endpoint to list all the characters. There are POST and GET endpoints to add and list greetings as well that we will write shortly.

<br />
<details>
  <summary><code>Character</code> routes </summary>

```python
from flask import Blueprint, request, abort, make_response
from ..db import db
from ..models.character import Character
from ..models.greeting import Greeting
from sqlalchemy import func, union, except_

bp = Blueprint("characters", __name__, url_prefix="/characters")

@bp.post("")
def create_character():

    request_body = request.get_json()
    try: 
        new_character = Character.from_dict(request_body)
        db.session.add(new_character)
        db.session.commit()

        return new_character.to_dict(), 201
    
    except KeyError as e:
        abort(make_response({"message": f"missing required value: {e}"}, 400))

@bp.get("")
def get_characters():
    character_query = db.select(Character)

    characters = db.session.scalars(character_query)
    response = []

    for character in characters:
        response.append(
            {
                "id" : character.id,
                "name" : character.name,
                "personality" : character.personality,
                "occupation" : character.occupation,
                "age" : character.age
            }
        )

    return response

@bp.get("/<char_id>/greetings")
def get_greetings(char_id):
    pass

@bp.post("/<char_id>/generate")
def add_greetings(char_id):
    pass

def validate_model(cls,id):
    try:
        id = int(id)
    except:
        response = {"message": f"{cls.__name__} {id} invalid"}
        abort(make_response(response , 400))

    query = db.select(cls).where(cls.id == id)
    model = db.session.scalar(query)
    if model:
        return model

    response = {"message": f"{cls.__name__} {id} not found"}
    abort(make_response(response, 404))
```
</details>
</br>

## Set up and connect the Database

Create and connect your database using the steps below:

1. Set up a development database in `psql` called `npc_generator_dev`
2. Create a `.env` file and add the database as your `SQLALCHEMY_DATABASE_URI`
3. Run `flask db init`, `flask db migrate` and `flask db upgrade` to connect the API to your database.
4. Test the POST and GET methods for `Character` in Postman.

<br />

<details>
  <summary>Expand for a sample request body to test the <code>Character</code> creation route in Postman</summary>

```JSON
{
    "name" : "Misty",
    "personality" : "Bubbly",
    "occupation" : "Gym Leader",
    "age" : 25
}

```
</details>
</br>

## Install the `google-genai` Library and Connect Your Secret Key

Before we write our request to the Gemini API within the project, we need to install the `google-genai` package. 

1. In your terminal, run `pip install -q -U google-genai`
2. Run `pip freeze > requirements.txt` to add the `google-genai` package and dependencies to your requirements.
3. In your `.env` file, add the variable `GEMINI_API_KEY` and assign it the value of your secret key: 
    `GEMINI_API_KEY=<Your Secret Key>`
4. In the `character_routes.py` file, import the `google-generativeai` package and configure the Gemini API environment using the code below:
   
    ```python
    from google import genai
    import os

    client = genai.Client()
    ```
   
   | <div style="min-width:300px;">Part of Code</div> | Description |
   | ------------- | ----------- | 
   | `from google import genai` | Import the `google-genai` package . |
   | `import os` | Import the `os` package so we can read our `GEMINI_API_KEY` from the `.env` file. |
   | `client = genai.Client()` | This code creates a Gemini API client with your secret key from the `.env` file. It looks specifically for a key named `GEMINI_API_KEY`. We will use the `genai` import to access the `gemini-2.5-flash` model and make requests to the `/gemini-2.5-flash:generateContent` endpoint. |

## Make the API call

To make an API call, we'll create a helper function called `generate_greetings` that will take a `Character` in as a parameter. The first thing we will do is construct a prompt for our request body that uses the `Character`'s attributes to describe what we are looking for. use the aliased `google.generativeai` import to get a reference to the AI model we want to use. To do this, we'll call the constructor for the `GenerativeModel` class and pass it the name of the model as an string argument `"gemini-1.5-flash"`.

```python
def generate_greetings(character):
    input_message = f"I am writing a fantasy RPG video game. I have an npc named {character.name} who is {character.age} years old. They are a {character.occupation} who has a {character.personality} personality. Please generate a Python style list of 10 stock phrases they might use when the main character talks to them. Please return just the list without a variable name and square brackets."
```
Once the prompt has been constructed we can use our `Client` object to send our Gemini API call. To access the desired `/gemini-2.5-flash:generateContent` endpoint we will call the `generate_content()` method on the `client` variable, passing our `input_message` as a parameter. The code we will use to make the call to the text completion endpoint and then store the response is:

```python
    response = client.models.generate_content(
        model="gemini-2.5-flash", contents=input_message
    )
```

Remember that the text completions endpoint returns a nested object. Let's pause here and recall what that object looks like. Using the example request data we shared to create the character Misty, we received the following response:

```JSON
{
    "candidates": [
        {
            "content": {
                "parts": [
                    {
                        "text": "\"Hey there, cutie! Ready to battle?\",\n\"Oh my gosh, I'm so excited to fight you!\",\n\"Let's have a super-duper fun battle!\",\n\"Prepare for the cutest, most awesome battle of your life!\",\n\"I hope you brought your A-game, because I'm bringing mine!\",\n\"Don't worry if you lose, it's all part of the fun!\",\n\"Wow, you're really strong! I love a good challenge!\",\n\"I'm gonna make this battle so sparkly!\",\n\"Ready to get your sparkle on?\",\n\"Let's go, let's go, let's go!\"\n"
                    }
                ],
                "role": "model"
            },
            "finishReason": "STOP",
            "avgLogprobs": -0.19814954297295931
        }
    ],
    "usageMetadata": {
        "promptTokenCount": 71,
        "candidatesTokenCount": 145,
        "totalTokenCount": 216
    },
    "modelVersion": "gemini-1.5-flash"
}
```

That is a lot of information but now it's up to us to parse through that response and determine what we want to use. Ultimately, we want to just use the `text` within `candidates` > `content` > `parts`. Happily for us, the `response` object returned by `generate_content` has a convenience attribute `.text` that we can use to directly access this `text` key within the JSON. Before we use this response in an endpoint, let's print out the text to see what we get:

```python
print(response.text)
```

This gives our `generate_greetings` helper method the final code of:

```python
def generate_greetings(character):
    input_message = f"I am writing a fantasy RPG video game. I have an npc named {character.name} who is {character.age} years old. They are a {character.occupation} who has a {character.personality} personality. Please generate a Python style list of 10 stock phrases they might use when the main character talks to them. Please return just the list without a variable name and square brackets."
    response = client.models.generate_content(
        model="gemini-2.5-flash", contents= input_message
    )
    print(response.text)
```

In order to test out what we get, we do still need to attach the `generate_greetings(character)` method to an endpoint. We can do this quickly by adding the following code to the `add_greetings(char_id)` function in our `/<char_id>/greetings` endpoint:

```python
@bp.post("/<char_id>/generate")
def add_greetings(char_id):
    character = validate_model(Character, char_id)
    generate_greetings(character)

    return character.to_dict()
```

This is not the final form our code for this endpoint will take, but if we make a post request to this endpoin in Postman using id 1, the terminal should display an AI generated response to our prompt! Go ahead and make this request now. If your terminal prints out the response, congratulations! You have successfully integrated an AI API into your backend! 

### !callout-info

## Knowing Your Response
Because our responses are AI generated, we have no real way of knowing exactly what will be returned until we make an API call. As a result, it is important to test out the response by making the call in Postman or printing the response. Knowing the shape of the response will make it much easier to pick and choose what information to use and how to format it correctly. In order to get more consistent responses, it may be a good idea to make your prompts more concise and error proof using shots or another of the prompting techniques we discussed earlier.

### !end-callout

## Using The Response

Now it's your turn. There are two endpoints left to write. First up is the POST request that creates "Greeting" records for a specific character by using the information returned from the call to the Gemini API. Try using what you know to write the `add_greetings` method. Don't forget to use the `generate_greetings` helper we just created.

<br />
<details>
  <summary> Try on your own and then compare with our solution </summary>

```python
@bp.post("/<char_id>/generate")
def add_greetings(char_id):
    character = validate_model(Character, char_id)
    greetings = generate_greetings(character)

    if character.greetings: # Check to see if Greetings have already been added
        return {"message": f"Greetings already generated for {character.name} "}, 201
    
    new_greetings = []
    
    for greeting in greetings:
        new_greeting = Greeting(greeting_text=greeting.strip("\","), # Strip leading and trailing quotes and commas from each greeting
                                character = character
                            )
        new_greetings.append(new_greeting)
    
    db.session.add_all(new_greetings)
    db.session.commit()

    return {"message": f"Greetings successfully added to {character.name}"}, 201
```
</details>
</br>

Once you've done that, try your hand at the `get_greetings` method as well.

<br />
<details>
  <summary> Try on your own and then compare with our solution </summary>

```python
@bp.get("/<char_id>/greetings")
def get_greetings(char_id):
    character = validate_model(Character, char_id)
    
    if not character.greetings:
        return {"message": f"No greetings found for {character.name} "}, 201
    
    response = {"Character Name" : character.name,
                "Greetings" : []}
    for greeting in character.greetings:
        response["Greetings"].append({
            "greeting" : greeting.greeting_text
        })
    
    return response
```
</details>
</br>

## Summary
In this lesson, we took a partially built API and expanded it to include AI generated content. This is just the tip of the iceberg when it comes to integrating the AI APIs into your code, so we encourage you as always to follow your curiosity and keep looking for new and exciting ways to bring AI into your projects!

### !callout-info

## Other AI APIs and Python
As mentioned before, Gemini is just one of many AI APIs you can integrate into your Python projects. We encourage you to explore others as your time and resources allow. While they won't look exactly the same, the steps for integration should be similar. There will likely be a Python package you can install with functions that will allow you to designate a model, attach your API key, make a request and more! Make sure to follow the documentation for whichever API you end up using and try things out!

### !end-callout  

## Check For Understanding

### !challenge
* type: tasklist
* id: 3b3735ed-2133-48fc-a54b-8a367e06c78c
* title: Check For Understanding
##### !question

Mark off each step that you have completed

##### !end-question
##### !options

* Set up NPC Generator
* Install the `google-generativeai` package
* Complete generate_greetings
* Complete add_greetings
* Complete get_greetings

##### !end-options
### !end-challenge
