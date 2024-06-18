# OpenAI API's Models

## Goals

Now that we have OpenAI set up on Postman, we can start taking a look under the hood at how OpenAI works and how we can integrate it into the programs we create. We will start by looking at the OpenAI API Models.

Our goals for this lesson are to:
- Recognize several commonly used OpenAI Models, their specific capabilities, and how they can be used.
- Understand some of the differences and limitations between the versions of GPT models.

## What is an OpenAI Model?

An OpenAI Model is a large language Model that used different data sets to perform Natural Language Processing. From answering questions to translating text, OpenAI has a variety of models that can perform different tasks depending on what your application needs. The original model upon which OpenAI was built was GPT-3. GPT stands for Generative Pre-Trained Transformer and GPT-3 was trained on 175 billion parameters. As OpenAI has grown, so have the models. GPT-3 has now been deprecated in favor of even larger models known as GPT-3.5 and GPT-4. That being said, OpenAI has a variety of models we can use depending on the needs of our projects:

| Model | Description | Use Case |
| ----- | ----------- | -------- |
| GPT-3.5 | A model that improves upon GPT-3 and is used primarily for Natural Language Processing. It can be used to understand and process natural language in order to generate code or respond to questions. | Chatbots, content generation |
| GPT-4/ GPT-4 Turbo | Models that further improve upon GPT-3.5. They perform the same functions but have been trained with even more data. They typically have a lower rate of hallucinations (but can still make mistakes!), and are considered one of the most powerful currently LLMs on the market. That being said, they are currently only available to use via both ChatGPT and the OpenAI API at an extra cost. | Chatbots, content generation |
| DALL-E | Not to be confused with our favorite trash collecting robot, DALL-E can be used to generate or edit an image or images via a Natural Language prompt. | Image generation, Image Manipulation | 
| TTS ("Text To Speech") | The TTS model converts text to natural sounding speech. When using the TTS model, the OpenAI API has a variety of pre-selected voices that can be used. These come in the form of a parameter within the API request. | Text to speech conversion | 
| Whisper | The Whisper model converts audio to text | Audio transcription | 
| Embeddings | The embeddings model converts text to its numerical equivalent in order to measure the "relatedness" of text strings. | Search queries, clustering text by similarity | 
| Moderation | The moderation model can be used to check and see if content might be considered sensitive, unsafe or unsuitable for certain populations. | Content moderation |  

If you are interested in learning about other models that are no longer widely used, OpenAI provides a list [here](https://platform.openai.com/docs/deprecations)  

### !callout-info

## Where are OpenAI Models Used? 
As we will see in the next lesson on endpoints, these models are most commonly used as query parameters within our API requests to tell OpenAI which model we would like to use when processing a specific request. 

### !end-callout

### !callout-info
  
## OpenAI Models and Pricing
  
The OpenAI API is not a free service at this point in time, so each model has a different pricing structure that depends on the size of the request that is being made. Pricing information can be found [here](https://openai.com/api/pricing)  

### !end-callout

## Summary
OpenAI provides models for everything from Text to Speech to Image Generation. We've also seen some ideas of applications that could incorporate the OpenAI models. If you are curious to know even more about the OpenAI models, don't hesitate to do more research in the [OpenAI API Documentation](https://platform.openai.com/docs/guides/text-generation)

## Check for Understanding 

### !challenge

* type: multiple-choice
* id: 143d8fd5-9070-4700-8ed0-58747edf997e
* title: Understanding OpenAI Models
<!-- * points: [1] (optional, the number of points for scoring as a checkpoint) -->
<!-- * topics: [python, pandas] (Checkpoints only. optional the topics for analyzing points) -->

##### !question

Which of the following models is used primarily for image generation and analysis?

##### !end-question

##### !options

a| DALL-E
b| TTS
c| Whisper
d| GPT-3.5 Turbo
e| GPT-4.0

##### !end-options

##### !answer

a|

##### !end-answer

### !end-challenge

