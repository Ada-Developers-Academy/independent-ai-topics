# Using the OpenAI API

## Goals

Now that we know a bit about the OpenAI endpoints, lets learn how to use them within our programs. To do so, we will work with a partially finished API that creates video game NPCs (Non Playable Characters). Within that program, we will install the OpenAI library and then leverage one of OpenAI's endpoints to generate dialogue for characters we create.

Our goals for this lesson are to:
- Integrate the OpenAI library into a python project.
- Use the OpenAI chat completion endpoint to generate content for our project. 

## Set Up the NPC Generator

This tutorial will walk you through how to integrate the OpenAI API into your projects with the companion repo [NPC Generator](https://github.com/AdaGold/NPC-Generator). We'll start with our usual setup.

1. Fork the repo to your github and then clone to your machine.
2. Create and run a virtual environment for the project.
3. Use `pip` to install the requirements.txt.

## Explore the NPC Generator repo

Take some time to explore the NPC Generator repo in its current state. For this project, we will be finishing an API for a role-playing style game. Our API creates NPCs (Non Playable Characters) that your player's character will interact with. 

We currently have two models, "Character" and "Greeting". "Character" holds some basic information about a character and has a one to many relationship with "Greeting", which represents a stock phrase that particular character may say when interacting with the player. Each NPC can have multiple greetings but each greeting can only be connected to one character. We will use the OpenAI API to generate those greetings which we will then store in our database.

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
from flask import Blueprint, jsonify, request, abort, make_response
from ..db import db
from ..models.character import Character
from ..models.greeting import Greeting
from sqlalchemy import func, union, except_
from openai import OpenAI

bp = Blueprint("characters", __name__, url_prefix="/characters")
client = OpenAI()

@bp.post("")
def create_character():

    request_body = request.get_json()
    try: 
        new_character = Character.from_dict(request_body)
        db.session.add(new_character)
        db.session.commit()

        return make_response(new_character.to_dict(), 201)
    
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

    return jsonify(response)

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
        response =  response = {"message": f"{cls.__name__} {id} invalid"}
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

Create and connect your database using the steps below

1. Set up a development database in `psql` called `npc_generator_dev`
2. Create a `.env` file and add the database as your `SQLALCHEMY_DATABASE_URI`
3. Run `flask init`, `flask migrate` and `flask upgrade` to connect the API to your database.
4. Test the POST and GET methods for `Character` in Postman.

<br />

<details>
  <summary>Sample Request Body for Testing in Postman</summary>

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

## Install the OpenAI Library and Connect Your Secret Key

Before we write our request to OpenAI within the project, we need to install the OpenAI Library. 

1. In your terminal, run `pip install --upgrade openai`
2. Run `pip freeze > requirements.txt` to add the OpenAI library to your requirements.
3. In your `.env` file, add the variable `OPENAI_API_KEY` and assign it the value of your secret key: `OPENAI_API_KEY=<Your Secret Key>`
4. In the `character_routes.py` file, import OpenAI and create an OpenAI environment using the code below:
    ```python

    from openai import OpenAI

    client = OpenAI()
    ```
| <div style="min-width:300px;">Part of Code</div> | Description |
| ------------- | ----------- | 
| `from openai import OpenAI` | Import the OpenAI library |
| `client = OpenAI()` | This code creates an OpenAI environment with your secret key from the `.env` file. We will use `client` to make requests to the OpenAI endpoints. |

## Make the API call

To make an API call, we'll create a helper function called `generate_greetings` that will take a `Character` in as a parameter. Then we'll use the `Character`'s attributes to construct a prompt for our request body. This may look something like:

```python
def generate_greetings(character):

    prompt = f"I am writing a video game in the style of The Witcher. I have an npc named {character.name} who is {character.age} years old. They are a {character.occupation} who has a {character.personality} personality. Please generate a python style list of 10 stock phrases they might use when the main character talks to them"

```
 Once the prompt has been constructed, we can insert it to the request body of our API call. The code we will use to make the call to the chat completion endpoint and then store the response is:

```python
completion = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "user", "content": <prompt>}
    ]
)

Remember that the chat completions endpoint returns a completion object. Let's pause here and recall what that object looks like:

```JSON
{
    "id": "chatcmpl-9bJ0xZYjN6QaI7OQ0UgrIatFwM98E",
    "object": "chat.completion",
    "created": 1718678255,
    "model": "gpt-3.5-turbo-0125",
    "choices": [
        {
            "index": 0,
            "message": {
                "role": "assistant",
                "content": "1. \"Hey there! Ready for a battle?\"\n2. \"Oh wow, you've come to challenge me? I'm so excited!\"\n3. \"Let's see what you've got! Bring it on!\"\n4. \"I'll give it my all! Let's make this battle intense!\"\n5. \"I hope you're prepared for my tough team of Pokémon.\"\n6. \"Don't underestimate me just because I'm bubbly. I'm a fierce competitor!\"\n7. \"I'm ready to show you the power of my gym leader skills!\"\n8. \"Win or lose, I always have a blast battling trainers like you.\"\n9. \"Let's have some fun and give it our all in this battle!\"\n10. \"No matter what happens, let's make this battle one to remember!\""
            },
            "logprobs": null,
            "finish_reason": "stop"
        }
    ],
    "usage": {
        "prompt_tokens": 72,
        "completion_tokens": 164,
        "total_tokens": 236
    },
    "system_fingerprint": null
}
```

That is a lot of information but now it's up to us to parse through that response and determine what we want to use. Ultimately, we want to just use the `content` within the `message`. We have a list of generated greetings, but they are all a single string. Fortunately, each item is separated by a newline character, so we can use that to our advantage. If we parse through the completion object and access the `content` string itself, we can split it by the newline character to return a list of responses. That return statement may look something like:
```python
return(completion.choices[0].message.content.split("\n"))
```

When we string this all together, our `generate_greetings` helper method is as follows:
```python
def generate_greetings(character):

    prompt = f"I am writing a video game in the style of The Witcher. I have an npc named {character.name} who is {character.age} years old. They are a {character.occupation} who has a {character.personality} personality. Please generate a python style list of 10 stock phrases they might use when the main character talks to them"
    completion = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "user", "content": prompt}
        ]
    )
    return(completion.choices[0].message.content.split("\n"))
```

### !callout-info

## Knowing Your Response
Because our responses are AI generated, we have no real way of knowing exactly what will be returned until we make an API call. As a result, it is important to test out the response by either printing it out or making the call in Postman. Knowing the response will make it much easier to pick and choose what information to use and how to format it correctly. In order to get more consistent responses, it may be a good idea to make your prompts more concise and error proof using shots or one of the other prompting techniques we discussed earlier.

### !end-callout

## Using The Response

Now it's your turn. There are two endpoints left to write. First up is the POST request that creates "Greeting" records for a specific character by using the information returned from the call to OpenAI. Try using what you know to write the `add_greetings` method. Don't forget to use the `generate_greetings` helper we just created.

<br />
<details>
  <summary> Try on your own and then compare with our solution </summary>

```python
@bp.post("/<char_id>/generate")
def add_greetings(char_id):
    character_obj = validate_model(Character, char_id)
    greetings = generate_greetings(character_obj)
    print(greetings)

    if character_obj.greetings:
        return make_response(jsonify(f"Greetings already generated for {character_obj.name} "), 201)
    
    new_greetings = []

    for greeting in greetings:
        text = greeting[greeting.find(" ")+1:] #Splice the numeric marker off each response
        new_greeting = Greeting(
            greeting_text = text.strip("\""), #Strip the response of any extraneous quotes
            character = character_obj
        )
        new_greetings.append(new_greeting)
    
    db.session.add_all(new_greetings) #Add all new Greeting records to db
    db.session.commit()

    return make_response(jsonify(f"Greetings successfully added to {character_obj.name}"), 201)
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
        return make_response(jsonify(f"No greetings found for {character.name} "), 201)
    
    response = {"Character Name" : character.name,
                "Greetings" : []}
    for greeting in character.greetings:
        response["Greetings"].append({
            "greeting" : greeting.greeting_text
        })
    
    return jsonify(response)
```
</details>
</br>

## Summary
In this lesson, we took a partially built API and expanded it to include AI generated content. This is just the tip of the iceberg when it comes to integrating the OpenAI API into your projects, so we encourage you as always to follow your curiosity and keep looking for new and exciting ways to bring AI into your projects!

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
* Install OpenAI Library
* Complete generate_greetings
* Complete add_greetings
* Complete get_greetings

##### !end-options

<!-- other optional sections -->
<!-- !hint - !end-hint (markdown, hidden, students click to view) -->
<!-- !rubric - !end-rubric (markdown, instructors can see while scoring a checkpoint) -->
<!-- !explanation - !end-explanation (markdown, students can see after answering correctly) -->

### !end-challenge

<!-- ======================= END CHALLENGE ======================= -->
