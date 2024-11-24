# More on AI Models

## Goals

Now that we can make calls to an AI API in Postman, we can start digging a little deeper into the AI model we are interacting with and how we might integrate it into the programs we create.

Our goals for this lesson are to:
- Get exposure to more applications of AI models. 
- Understand key features and limitations of the Google Gemini 1.5 Flash model at the free tier of service.

## What is an AI Model?

An AI Model is a large language Model that uses different data sets to perform Natural Language Processing. From answering questions to translating text, organizations that create and host AI APIs have a variety of models that can perform different tasks depending on what your application needs. Some common applications that AI APIs may have models for include:

| Description | Use Case |
| ----------- | -------- | 
| Natural Language Processing, which can be used to understand and process natural language like free form text or a document in order to generate code or respond to questions. | Chatbots, content generation |
| Generating or editing an image or images via a Natural Language prompt. | Image generation, Image Manipulation |
| Converting text to natural sounding speech.| Text to speech conversion | 
| Converting audio to text or analyzing an audio clip guided by a text prompt. | Audio transcription, Audio descriptions, Captioning | 
| Analyzing a video, image, or images guided by a text prompt. | Image or video descriptions, Image content analysis |

As an example, the original model upon which OpenAI was built was GPT-3. GPT stands for Generative Pre-Trained Transformer and GPT-3 was trained on 175 billion parameters. GPT-3 has now been deprecated in favor of even larger models known as GPT-3.5 and GPT-4. 

When choosing a model to work with we can start by asking ourselves, what kind of output do we need? Are we looking for text, audio, images, etc? That will narrow down which API providers could meet our needs. At that point we need to look at each model's limitations and pricing to see what fits both our budget and the kind of traffic we think we'll need to support for our feature. Most free tier limits are great for testing things out, but not scalable as a permanent solution if an AI feature becomes popular.  

### !callout-info

## Where is an AI Model Used? 
These models are most commonly used as query parameters within our API requests to tell the AI API which model we would like to access when processing a specific request. 

### !end-callout
 
## Google Gemini 1.5 Flash
  
While we work with the Google Gemini API, we will be using the Gemini Flash 1.5 model specifically. We are using this model because it's free tier fits our needs to show off some text-based use cases, while allowing for exploration past what we will cover in class for folks who are interested. We will cover facts we think are useful below, for more information, check out the [Google Gemini API Documentation](https://ai.google.dev/gemini-api/docs/models/gemini).

**Gemini 1.5 Flash Model Facts**
| Category | Detail |
| -------- | ------ | 
| Input Types | Text, Audio, Images, Video |
| Output Types | Text |
| Maximum Requests Per Minute | 15 Requests Per Minute |
| Maximum Tokens Per Minute | 1 million Tokens Per Minute |
| Maximum Requests Per Day | 1,500 Requests Per Day |
| Supported Languages | 30+, see documentation for details |

When thinking about tokens and our processing limits, a token is equivalent to about 4 characters for Gemini models. 100 tokens are about 60-80 English words. Token limits can creep up on us if we're doing a significant amount of work. A model needs to process the data we send it into tokens, and the model will put together tokens to create its text response, all of which count towards our total processing limits. If we are working and suddenly see requests failing, we may need to look at our usage and possibly wait for a bit until we can make requests again if we are out of processing resources.

## Summary
AI APIs provide models for everything from Text to Speech to Image Generation. We've also seen some ideas of applications that could incorporate various models. If you are curious to know even more about the Gemini models, don't hesitate to do more research in the [Google Gemini API Documentation](https://ai.google.dev/gemini-api/docs/models/gemini)!

## Check for Understanding 

### !challenge

* type: multiple-choice
* id: 143d8fd5-9070-4700-8ed0-58747edf997e
* title: More on AI Models

##### !question

Which of these actions typically contributes to token usage for an AI API? 
Select one option.

##### !end-question

##### !options

a| Sending text to a model to process
b| Sending audio to a model to process
c| Sending images to a model to process
d| Sending video to a model to process
e| A model creating a response
f| All of the above

##### !end-options

##### !answer

f|

##### !end-answer

### !end-challenge

