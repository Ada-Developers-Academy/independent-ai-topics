# Using the OpenAI API

## Goals

Now that we know a bit about the OpenAI endpoints, lets learn how to use them within our programs. To do so, we will work with a partially finished API that creates video game NPCs (Non Playable Characters). Within that program, we will install the OpenAI library and then leverage one of OpenAI's endpoints to generate dialogue for characters we create.

Our goals for this lesson are to:
- Integrate the OpenAI library into a python project.
- Use the OpenAI chat completion endpoint to generate content for our project. 

## NPC Generator Setup

This tutorial will walk you through how to integrate the OpenAI API into your projects with the companion repo [NPC Generator](https://github.com/AdaGold/NPC-Generator). We'll start with our usual setup.

1. Fork the repo to your github and then clone to your machine.
2. Create and run a virtual environment for the project.
3. Use pip to install the requirements.txt.

## Exploring the NPC Generator

Take a second to explore the NPC Generator in its current state. For this project, we will be finishing an API that creates NPCs (Non Playable Characters) for a Witcher style game. We currently have two models, "Character" and "Greeting". "Character" holds some basic information about a character and has a one to many relationship with "Greeting", which represents a stock phrase that particular character may say when interacting with the player. Each NPC can have multiple greetings but each greeting can only be connected to one character. We will use the OpenAI API to generate those greetings which we will then store in our database.

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

Now let's take a look at the routes we currently have. If you examine the code below, we currently have our validate_model helper method as well as a POST endpoint to add a character and a GET endpoint to list all the characters. There are POST and GET endpoints to add and list greetings as well that we will write shortly.

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

## Setting up and connecting the Database

1. Set up a development database in psql called npc_generator_dev
2. Create a .env file and add the database as your `SQLALCHEMY_DATABASE_URI`
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
