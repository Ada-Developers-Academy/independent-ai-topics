# Introduction to AI APIs

## Goals

We explored how we can use tools built on Large Language Models like ChatGPT to generate ideas and to debug and review code, but what if we wanted to leverage something like Google Gemini or ChatGPT itself in our projects? In this tile, we'll learn about AI APIs - the interfaces we can interact with that power tools like ChatGPT - and how to set them up in a way that brings AI and LLM's many powerful tools right to our doorstep.

Our goals for this lesson are to:
- Understand what AI APIs are and how we can use them
- Understand how AI APIs are different than customized tools like ChatGPT

## Vocabulary

| <div style="min-width:100px;">Vocab</div> | Definition |
| ----- | ---------- | 
| AI Model | An AI program that has been trained on data sets to achieve a specific purpose. Depending on their purpose, generative AI models are able to create content from different types of data input, including text, images, and audio. At their most basic level, generative models operate like sophisticated autocomplete applications. |
| AI API | An API that allows users to access and deploy capabilities of AI models through specific endpoints so they can enable AI features within their own applications. Several AI APIs exist, some of the more well known ones are: the [OpenAI API](https://platform.openai.com/docs/api-reference/introduction), [Google Gemini API](https://ai.google.dev/gemini-api/docs), and [Meta's Llama API](https://llama.developer.meta.com/docs/overview/). |

## What is an AI API?

Just as an API allows us to interact with and access the functions, data and other features of a particular service, an AI API allows us to access the features provided by the AI models of a particular organization. As a result, the AI APIs available can be useful for integrating a variety of AI tools into the applications we create. 

Some practical applications include but are not limited to:
- Creating a chatbot or virtual assistant
- Content generation from user input
- Image recognition
- Language translation
- Generating recommendations based off user input
- Providing accessibility considerations such as text to speech or speech to text

Every AI API is going to be a little different in what it offers. Nearly all have some kind of text generation, image processing, and audio processing, but the limits of what exactly you can do with these will vary based on the API you're using. When you want to build an AI enabled feature, an early part of planning will be researching what is currently available from AI APIs and determining which of the APIs will support the needs of your application. 

How you will interact with the API may also depend on language you're using or the platform you're programming for. AI APIs can be reached through RESTful endpoints, but some organizations may also publish language-specific libraries to make their AI offerings a little easier to work with.

## AI APIs vs Customized Products

In order for us to work all of these tools into our projects, it is important to understand the distinction between customized products ready for public consumption like ChatGPT and the APIs that power these tools like the OpenAI API itself. As we have discussed before, ChatGPT is an online platform that is open for use to the general public. It is a (mostly) free service and it is primarily used for generating text, retrieving information and engaging in dialogue with the LLM on which it has been trained.

On the other hand, the OpenAI API is an API that allows developers to integrate their GPT models into their applications. Similarly the Gemini API allows developers to integrate the Gemini family of models into their projects. We will discuss working with text models further in a following lesson, but briefly, models are LLMs that have been trained for specific actions, whether that be retrieving information or generating content, etc. AI APIs allow us to access these different models and use them in our development.

Different model versions can even be used within the same application. Taking ChatGPT as an example, in the past, the free version was built using a slightly older 3.0 GPT model from the OpenAI API. It was built specifically to answer questions, provide recommendations, or generate content based off user text input. At the time, the paid version was built similarly but used the better trained 4.0 GPT model for its conversational piece and also incorporated other models that could handle input in the form of images, video or audio. 

If we wanted to, we could use an AI API to access a generative text model to build our own pseudo ChatGPT, but since that has already been created, it is up to us to find other exciting ways to leverage AI APIs within our own projects!

### !callout-warn

## Many AI APIs are Not Free at this Time
Unfortunately, some APIs such as the OpenAI API do not currently have a free tier or credits for exploration. Many AI APIs are priced according to which model you are using and how often you are using it. This information is accurate as of the time this lesson has been published. AI services are evolving rapidly and offerings may have changed at the time of reading.

### !end-callout

## Summary
In this lesson, we've learned the difference between AI APIs and customized products built using AI like Canva's AI image generation and Microsoft's Copilot. In the next lessons, we will take a look at how to work with an AI API and start integrating AI functionality into our projects!