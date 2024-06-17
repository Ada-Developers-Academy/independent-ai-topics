# Using the OpenAI API

## Goals

Now that we know a bit about the OpenAI models, lets learn how to use them within our programs. To do so, we'll start by exploring OpenAI endpoints. From there we will work with a partially finished API that creates video game NPCs (Non Playable Characters). Within that program, we will install the OpenAI library and then leverage one of OpenAI's endpoints to generate dialogue for characters we create.

Our goals for this lesson are to:
- Recognize different OpenAI endpoints and how they are used
- Understand how to integrate the OpenAI library into a python project.
- Use the OpenAI chat completion endpoint to generate content for our project. 

## Exploring OpenAI Endpoints

In the last lesson, we looked at the different models associated with OpenAI. It may come as no surprise that each model maps to a specific endpoint we can use to access different AI functionalities. Below is a non-exhaustive list of OpenAI's endpoints and the models they map to:

| Endpoint | Model(s) | Usage |
| -------- | -------- | ----- |
| POST https://api.openai.com/v1/chat/completions | GPT 3.5 Turbo/GPT 4.0/ GPT 4.0 Turbo | Generating a response to a set of messages |
| POST https://api.openai.com/v1/images/generations | DALL-E | Generate a new image from a given prompt |
| POST https://api.openai.com/v1/audio/speech | TTS | Generate audio from input |
| POST https://api.openai.com/v1/audio/transcriptions | Whisper | Generate a transcription of audio input |
| POST https://api.openai.com/v1/embeddings | Embeddings | Creates a numerical  representation of text |
| POST https://api.openai.com/v1/moderations | Moderations | Checks to see if text could be potentially harmful | 
| GET https://api.openai.com/v1/models | None | Returns a list of OpenAI models | 

We will focus on using the chat completions endpoint for now, but  each endpoint can be called similarly. Each request body will require some form of input at a minimum and most will require the specific model you would like to use as well. The response you receive will also vary from request to request so make sure you have an understanding of the shape of a response before jumping right in. The full list of OpenAI endpoints along with examples of a request and response body can be found [here](https://platform.openai.com/docs/api-reference/introduction). OpenAI also provides a [model compatability chart](https://platform.openai.com/docs/models/model-endpoint-compatibility) that allows you to see which models work with each endpoint.

## The Chat Completions Endpoint

Before we jump right into integrating the API into a project, let's explore the chat completions endpoint. This particular endpoint is used to mimic and then continue a conversation between the user and an artificial intelligence. In terms of chatGPT, the chat completion endpoint is what is used to interact with the GPT models. The term "chat completion" simply refers to the fact that the response will be the next logical reply to the conversation that is going on. 

### Chat Completion Request Body

As mentioned earlier, the chat completion endpoint is: 
```
    POST https://api.openai.com/v1/chat/completions 
``` 
Like most POST endpoints, the chat completions endpoint is not complete itself without a request body. This particular endpoint has two required properties:

- messages - A list of messages that hold the conversation so far.
- model - The model to be used. As mentioned earlier, GPT 4.0 might return a different response than GPT 3.5.

We encourage you to research all the opther optional properties for this endpoint but some of note are:

- max-tokens - The maximum number of tokens to be returned
- n - The number of choices you'd like the api to return. The default value is 1. 
- temperature - A sampling temperature from 0-2. The higher the value, the more random the response. The default value is 1.

The rest of the optional properties can be found [here](https://platform.openai.com/docs/api-reference/chat/create).


Now that we know what is required, let's test the chat completions endpoint in Postman. Go ahead and open up Postman and enter the POST endpoint from above. Make sure you are in your OpenAI environment and then navigate to set up a raw JSON body. In the window, copy and paste the request body below.
```
{
    "model": "gpt-3.5-turbo",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "Hello! Are you able to understand this?"
      }
    ]
}
```

### !callout-info

## The Message Object

You may notice that each message object in our request body has two properties: "role" and "content". The "role" property indicates who the message came from. For our purposes, we have two options, "user" and "system". The content property holds the message text itself. Both properties are required to make a valid request.

### !end-callout

Go ahead and run the request and then let's check out the response body.

### Chat Completion Response Body

After running the above request, we received the response below. Your response may look different, but the properties should be the same or similar. This response is known as a chat completions object.
```
{
    "id": "chatcmpl-9bBybziJstOHt9FAWJX8MOWA8QJ2B",
    "object": "chat.completion",
    "created": 1718651201,
    "model": "gpt-3.5-turbo-0125",
    "choices": [
        {
            "index": 0,
            "message": {
                "role": "assistant",
                "content": "Yes, I can understand your message. How can I assist you today?"
            },
            "logprobs": null,
            "finish_reason": "stop"
        }
    ],
    "usage": {
        "prompt_tokens": 26,
        "completion_tokens": 15,
        "total_tokens": 41
    },
    "system_fingerprint": null
}
```
A few of the response properties to take note of are:
- id - A unique identifier for the response.
- model - The model you used.
- choices- A list of potential responses that you can choose from. If you did not specify an n in the request body, there will only be one choice.
- usage - Details on how many tokens were used in the request and response. 

You have now written your first request to the OpenAI API! 

## Summary
We should now know the different endpoints available to us via the OpenAI API. We should also be able to identify required properties for the chat completion endpoint. Feel free to play around with different request bodies and see how the response changes!

## Check for Understanding

### !challenge

* type: checkbox
* id: ad3cb5d0-fe7b-4a4d-96df-c7c94f9f111b
* title: Understanding OpenAI Endpoints

##### !question

Which of the following request body properties are REQUIRED for a chat completion POST request?

##### !end-question

##### !options

a| n
b| max-tokens
c| messages
d| model
e| user 

##### !end-options

##### !answer

c|
d|

##### !end-answer

#### !explanation

Only messages and model are required. When writing your messages however, make sure they include both role and content as properties!

#### !end-explanation

### !end-challenge

