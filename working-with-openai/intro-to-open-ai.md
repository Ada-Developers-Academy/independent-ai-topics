# Introduction to the OpenAI API

## Goals

We explored how we can use tools built on OpenAI's offerings like ChatGPT to generate ideas and to debug and review code, but what if we wanted to leverage ChatGPT itself in our projects? In this lesson, we'll learn about the OpenAI API - the API behind ChatGPT itself - and how to set it up in a way that brings ChatGPT's many powerful tools right to our doorstep.

Our goals for this lesson are to:
- Understand what the OpenAI API is and how we can use it
- Understand how the OpenAI API is different than ChatGPT

## Vocabulary

| Vocab | Definition |
| ----- | ---------- | 
| OpenAI | OpenAI is an organization that specializes in Artificial Intelligence research and deployment |
| OpenAI API | An API created by OpenAI to allow users to access and deploy AI models within their own applications. |
| OpenAI Model | An AI program that has been trained on OpenAI data sets to achieve a specific purpose. |

## What is the OpenAI API?

Just as an API allows us to interact with and access the functions, data and other features of a particular service, the OpenAI API allows us to access the features provided by ChatGPT's parent organization, OpenAI. As a result, the OpenAI API can be useful in integrating a variey of AI tools into the applications we create. 

Some practical applications include but are not limited to:
- Creating a chatbot or virtual assistant
- Content generation
- Image recognition
- Language translation
- Generating recommendations based off user input
- Providing accessibility considerations such as text to speech or speech to text

## The OpenAI API vs ChatGPT

In order for us to work all of these tools into our projects, it is important to understand the distinction between ChatGPT and the OpenAI API itself. As we have discussed before, ChatGPT is an online platform that is open for use to the general public. It is a (mostly) free service and it is primarily used for generating text, retrieving information and engaging in dialogue with the LLM on which it has been trained.

On the other hand, the OpenAI API is an API that allows developers to integrate GPT models into their applications. We will discuss GPT models in a bit, but briefly, they are LLMs that have been trained for specific actions, whether that be retrieving information or generating content, etc. The OpenAI API allows us to access these different models and use them in our development.

If we look at ChatGPT itself, we see that the free version is built using the GPT 3.5 model from the OpenAI API. It is built specifically to answer questions, provide recommendations, or generate content based off user text input. The paid version is built similarly but uses the better trained GPT 4.0 model for its conversational piece and also incorporates other models that can handle input in the form of images, video or audio. If we wanted to, we could use the OpenAI API to build our own pseudo ChatGPT, but since that has already been created, it is up to us to find other exciting ways to leverage OpenAI's API within our own projects.

## Setting up the OpenAI API

### !callout-warn

## The OpenAI is Not Free at this Time
Unfortunately, the OpenAI API is not currently a free API. It is priced according to which model you are using and how often you are using it. New users do currently get $5 worth of credits to begin experimenting which is tied to the phone number you provide when signing up. This information is accurate as of the time this lesson has been published. The OpenAI services are changing rapidly and offerings may have changed at the time of reading.


### !end-callout