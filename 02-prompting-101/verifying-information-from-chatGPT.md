# Verifying Information from ChatGPT

## Goals

There are myriad uses for Generative AIs where we want them to be creative and come up with new things that don’t exist in the world, but what do we do when it really matters that a response is concrete, factual information? 

Our goals for this lesson are to:
- Name specific difficulties with trusting information from Generative AIs
- Review techniques for searching and validating information

## Recap: Issues with Generative AI Responses
The Generative AI overview readings touched on some of the issues below, but let’s remind ourselves of some of the facts about LLMs which we need to be aware of when using them or tools they power.

### Hallucinations 
In generative AIs, hallucinations are when they make up information that is partially or fully inaccurate. Older LLMs are typically more prone to fabricating information, though newer ones can and still do. OpenAI shared the following with respect to how their newer model GPT-4 fares compared to GPT-3.5:

> “We spent 6 months making GPT-4 safer and more aligned. GPT-4 is 82% less likely to respond to requests for disallowed content and 40% more likely to produce factual responses than GPT-3.5 on our internal evaluations.”   [Source](https://openai.com/gpt-4 )

### Stale information 

The information that tools like ChatGPT provides is based on the data they were trained on. When new events unfold or information comes to light that contradicts what was in the data set at the time the AI was being trained, these AI tool's responses can be inaccurate or contain outdated information. 

### Citing sources

An AI doesn’t know where its training data came from to be able to cite where facts came from. This is changing, but at the moment, Generative AIs don’t typically have internet access to look up information. 

Newer tools that augment generative AI with search capabilities such as “Bing Chat” are trying to increase their reliability by grounding their answers in external sources of truth (tactics like this can also help deal with the knowledge cutoff of their training data). However, a website cited by a search augmented AI tool can still contain misinformation so these augmented tools should not be seen as a solution that prevents needing to fact check. We still need to confirm that the sources used are trustworthy and accurate, but it does at least give us a starting point for our fact checking. 

The issue of generating non-existent citations is widespread enough that some universities have statements [like this one from University of Arizona](https://ask.library.arizona.edu/faq/387173) on what to do when someone can't find the citations ChatGPT provided.

### Inclusion of information from biased sources

Not all sources should be given the same credence. A resource or article that a tool mentions actually existing does not mean that it’s a trustworthy source of information. There can be a vast amount of biased information in the datasets which LLMs are trained on, meaning we need to take a critical lens to the responses AIs generate to ensure we aren’t propagating harmful biases using what they’ve generated.

## How Can We Validate?

So, if the AI tool doesn’t know the latest news, can’t tell us where its training data came from, or it isn’t capable of looking the facts up online itself, how can we go about checking information in the response?

Fact checking! We’ll use our searching powers that we’ve honed looking up documentation and tutorials to take a critical eye to any information in the response that can be verified such as:
- proper names
- place names
- references to time, distance, date, season
- physical descriptions
- references to the gender of anyone described 
- quotations (and facts within quotes)
- any argument or narrative that depends on fact

We want to cross-check details of the response with other reliable sources of information. We may need to do several searches, and our fact checking could include consulting multiple reputable websites, academic publications, or experts in the topic. Depending on the subject matter we can pull in other resources like databases of news and journal articles (such as LexisNexis or ScienceDirect), and tools like Google Scholar.

When a generative AI tool makes a claim like "Ida B. Wells became editor and co-owner of the newspaper 'The Free Speech and Headlight' in 1889" we can look for multiple sources that confirm whether: 
- Ida B. Wells existed
- The newspaper 'The Free Speech and Headlight' existed
- If she became the editor and co-owner in 1889 

### Looking for SMART resources

When doing our own searching, we also need to be asking ourselves if the sources we’re using to verify information are trustworthy. Anyone can publish anything on the internet, but there are frameworks we can use to ask ourselves questions about a website or source to help us investigate if it may be worthwhile. One such framework is the SMART Check:

| Abbreviation | Meaning | Questions to ask |
| ------------ | ------- | ---------------- |
| S | Scope | What is the scope of the article? Does it cover the subject with enough depth and detail? |
| M | Motivation | What is the purpose of the article? To sell you an idea or to inform? |
| A | Authority & Accuracy | Who created the article, and what are the author's credentials? Can the information be verified? |
| R | Relevancy & Reliability | Is the content appropriate for your needs? Do you have to have scholarly sources? |
| T | Timely | Is the content current? Is the site still regularly maintained? |

For more information on questions you can ask for each part of the acronym, [check out this resource](https://uscupstate.libguides.com/news_aware/SmartTest) from University of South Carolina Upstate.

This need to validate responses leads us to another important takeaway: there are some tasks and questions where generative AI tools simply aren’t useful. For example, if we want to know whether it’s safe for a pet to eat a particular food, we can ask a tool like ChatGPT, but we’re still going to need to verify that the response it gave us is accurate. We can search for that information from a trustworthy source, like a well established website on pet caretaking and save ourselves some time by cutting out the middle step of asking ChatGPT. If we want a definite yes or no about if a specific event happened or a specific fact is true, current generative AI tools cannot answer that with 100% certainty, even if their response is written as though it is confident. 

## Optional Fact Checking Resources

If you'd like to dive more into fact checking and sharpen your skills, these are some further resources that expand on what we've shared:
- [CUNY - Fact Check Your Work](https://researchguides.journalism.cuny.edu/factchecking-verification/fact-check-your-work)
- [LexisNexis - How to fact check like a pro](http://www.lexisnexis.com/pdf/nexis/Nexis-webinar-how-to-fact-check-like-a-pro.pdf)
- [Ohio State University Pressbooks - Choosing and Using Sources: Fact or Opinion](https://ohiostate.pressbooks.pub/choosingsources/chapter/fact-or-opinion/)
- [Grove City College - Should I Use This Resource graphic](https://hbl.gcc.libguides.com/research/credible)

## Summary 

Generative AI tools are very powerful, but at this stage, there are several issues around hallucinations, the cutoff date of their training data, inability to connect to the internet to gather new or further information, and inability to share sources from their training data, which together mean that we cannot trust any facts that they state without verification. 

Any time we ask an AI tool for concrete information about the real world: historical dates, people, places, events, programming languages, etc., we need to break out our web searching and fact-checking skills to confirm each piece of verifiable information. 

While we verify information, we need to be checking if the sources we are using are credible by looking at who is writing the information, why they are writing it, who is the intended audience, and where the author’s information is coming from. If we do not take care with validating facts before sharing information from Generative AIs, we may be responsible for putting more misinformation out into the world that could be further used to train LLMs or be incorrectly cited by others. 

## Check for Understanding

<!-- Question 1 -->
<!-- prettier-ignore-start -->
### !challenge
* type: checkbox
* id: 4f5ad7fa-859f-11ee-b9d1-0242ac120002
* title: Verifiable Information

##### !question
What kinds of verifiable information are present in the following statement?

> Born in 1918, Katherine Johnson, a pioneering mathematician and physicist, played a crucial role in the early years of the United States' space program. Her mathematical expertise was instrumental in calculating trajectories for historic space missions, including the first American in space and the Apollo 11 moon landing. Johnson's groundbreaking work, along with her colleagues at NASA, helped propel the nation into the space age and challenge the prevailing stereotypes of women and African Americans in STEM fields. Her contributions were recognized in 2015 when she was awarded the Presidential Medal of Freedom, and her legacy continues to inspire future generations of scientists and mathematicians.

##### !end-question

##### !options
* proper names
* place names
* references to time, distance, date, season
* physical descriptions
* references to the gender of anyone described 
* quotations (or facts within quotes)
* an argument or narrative that depends on fact
##### !end-options

##### !answer
* proper names
* place names
* references to time, distance, date, season
* references to the gender of anyone described 
##### !end-answer

### !end-challenge
<!-- prettier-ignore-end -->

<!-- Question 2 -->
<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: 4f5adc96-859f-11ee-b9d1-0242ac120002
* title: Using SMART

##### !question
What part of the SMART check should give me pause about the statement below?

> I found a website that supports the fact I’m checking, but the site hasn’t been updated since it was first created 5 years ago. 

##### !end-question

##### !options
* S - Scope
* M - Motivation
* A - Authority & Accuracy
* R - Relevancy & Reliability
* T - Timely
##### !end-options

##### !answer
* T - Timely
##### !end-answer

### !end-challenge
<!-- prettier-ignore-end -->