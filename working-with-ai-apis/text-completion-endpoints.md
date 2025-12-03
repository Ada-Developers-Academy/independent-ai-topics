# Text Completion Endpoint Requests and responses

## Goals

Now that we know a little about AI models, let's learn about the text completion endpoint we will be using. We will be working with the `generateContent` method provided by the `gemini-2.5-flash` model.

Our goals for this lesson are to:
- Learn about the structure of requests and responses to Google Gemini's `/v1beta/models/gemini-2.5-flash:generateContent` endpoint

## Anatomy of an Endpoint

The full URL that we have been using to make `POST` requests from Postman is:
```
https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$GEMINI_API_KEY
```
From our prior learning, we should be able to recognize different parts of the URL, even if we don't know exactly what they're doing. Let's take a closer look at each part of the URL:
|  <div style="min-width:200px;">Part of URL</div>  | Purpose |
| ----------- | ------- |
| `generativelanguage.googleapis.com` | The domain hosting the API |
| `/v1beta` | Used to identify the version of the API being used |
| `/models` | Identifies that we want to access a `models` resource |
| `/gemini-2.5-flash` | Names that we are specifically requesting the `gemini-2.5-flash` model |
| `:generateContent` | States the function of the specified model that we would like to invoke |
| `?key=$GEMINI_API_KEY"` | Query parameter named `key` with a value containing our Google Gemini API key to authenticate our request |

We will be focusing solely on using the `/gemini-2.5-flash:generateContent` endpoint and working with text input and output. Please feel free to follow your curiosity and check out the API documentation to explore other kinds of input you can use to create more complex features. Make sure you have an understanding of the shape of the request and response before jumping right in. Test things out in Postman if it helps!

## Text Completion Requests and Responses

A text completion endpoint is used to mimic and then continue a piece of text or conversation between the user and an artificial intelligence. In terms of tools like ChatGPT, a text completion endpoint is what is used to interact with a model to generate more content. The term "text completion" simply refers to the fact that the response will be the next logical text to continue the conversation that is going on. 

### Text Completion Request Body

As mentioned earlier, the `/gemini-2.5-flash:generateContent` text completion endpoint takes a POST request at the URL:
`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$GOOGLE_API_KEY`

Like most POST endpoints, a request to the text completions endpoint is not complete itself without a request body. Let's go ahead and write a request body and then discuss its individual parts.

```
{
  "contents": [
    {
      "parts": [
        {
          "text": "Tell me about Ada Lovelace"
        }
      ]
    }
  ]
}
```

Now let's look at each piece a little more in depth

|  <div style="min-width:150px;">Property</div> | Description |
| ------------- | ----- | 
| `contents` | The list of input data and associated metadata sent to the model |
| `parts` | For a single entry in `contents`, the `parts` key will contain all the input data for a specific message to the model. The value for this key could have multiple entries if we were also sending other types of input like images or audio. |
| `text` | Any textual input to the model is sent as a value for this key. |

The 3 properties above are REQUIRED for your request to work properly. Each entry of `contents` can have a `role` property of `user` or `model` that can be useful when building a chat history or any other application where you want to collect multiple rounds of requests and responses for context. An example of the body data for sending a history with multiple messages might look like:
```
{
  "contents": [
    {
      "role": "user",
      "parts": [
        {
          "text": "Hi"
        }
      ]
    },
    {
      "role": "model",
      "parts": [
        {
          "text": "Nice to meet you. How can I help?"
        }
      ]
    },
    {
      "role": "user",
      "parts": [ 
        {
          "text": "I have three cats in my house. How many paws are in my home?"
        }
      ]
    }
  ]
}
```

Every prompt we send to a model includes parameters that control how the model generates responses. If we don't explicitly configure those parameters, the model will use default options, which can vary by model. For many applications, and now while testing things out, the defaults are perfectly fine. If you'd like to change the response generation settings, we can include a `generationConfig` key in our body JSON data to set the properties below. The `generationConfig` and the settings that follow are all OPTIONAL. 

| <div style="min-width:175px;">Property</div> | Description |
| -------- | ----------- | 
| `stopSequences` | specifies the set of character sequences (up to 5) that will stop output generation. If specified, the API will stop at the first appearance of a stop_sequence. The stop sequence won't be included as part of the response. |
| `temperature` | controls the randomness of the output. Use higher values for more creative responses, and lower values for more deterministic responses. Values can range from [0.0, 2.0]. The default value is 1. |
| `maxOutputTokens` | Sets the maximum number of tokens to include in a candidate. |
| `topP` | changes how the model selects tokens for output. Tokens are selected from the most to least probable until the sum of their probabilities equals the `topP` value. The default `topP` value is 0.95. |
| `topK` | changes how the model selects tokens for output. A `topK` of 1 means the selected token is the most probable among all the tokens in the model's vocabulary, while a `topK` of 3 means that the next token is selected from among the 3 most probable using the temperature. Tokens are further filtered based on `topP` with the final token selected using temperature sampling. |

For more information and examples, check out the ["Configure text generation"](https://ai.google.dev/gemini-api/docs/text-generation?lang=rest#configure) section of the Text Generation Gemini API documentation.

Now, let's open up Postman, enter the POST endpoint from above, and set the `x-goog-api-key` API key authorization value to our Google Gemini API key. Once that is ready, navigate to set a raw JSON body. In the window, copy and paste the conversation body example from above, then run the request so we can check out the response body!

### Text Completion Response Body

After running the above request, we received the response below. Your response may look different, but the properties should be the same or similar. 

```
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "Each cat has 4 paws, and you have 3 cats, so there are 3 * 4 = 12 paws in your home.\n"
          }
        ],
        "role": "model"
      },
      "finishReason": "STOP",
      "avgLogprobs": -0.006260416004806757
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 30,
    "candidatesTokenCount": 32,
    "totalTokenCount": 62
  },
  "modelVersion": "gemini-2.5-flash"
}
```

| <div style="min-width:150px;">Piece of Code</div> | Notes |
| ------------- | ----- | 
| `candidates` | A list of potential responses that you can choose from. There are changes around this underway but typically there will only be one choice. |
| `modelVersion` | The model that was used |
| `usageMetadata` | Details on how many tokens were used in the request and response |

You have now written your first request using a history of messages to the Gemini API! 

To read through the API's generated response to our prompt, we will need to: 
1. iterate through the `candidates` list 
2. for each element in `candidates` access the `content` key  
3. access the `parts` key and iterate over the content's `parts` list 
4. for each element in `parts` read the `text` response and handle that string however we would like.  

This might sound a little tedious, but in our upcoming case study we'll show how we can use Google's `[google-genai](https://pypi.org/project/google-genai/)` Python package to make accessing the response text a little easier.

## Summary
We should now know the expected shape of requests and responses using Google's Gemini API. We should be able to identify required properties for a text completion endpoint. Feel free to play around with different request bodies and see how the response changes!

## Check for Understanding

### !challenge

* type: checkbox
* id: ad3cb5d0-fe7b-4a4d-96df-c7c94f9f111b
* title: Text Completion Endpoint Requests and responses

##### !question

Which of the following request body properties are always required for a POST request to `/gemini-2.5-flash:generateContent`?

##### !end-question

##### !options

a| temperature
b| role
c| contents
d| generationConfig
e| text 

##### !end-options

##### !answer

c|
e|

##### !end-answer

#### !explanation

Of the properties listed, only `contents` and `text` are always required. `role` is necessary when we are sending multiple messages and need to distinguish context. `generationConfig` is always optional and we only need to include it if we are changing default response generation settings like `temperature`. 

#### !end-explanation

### !end-challenge

