# Setting up Our Workspace

We'll be using two tools to provide free access to large language models for coding agents.
- [Github Copilot](https://github.com/features/copilot), through the GitHub Student Developer Pack
- [OpenRouter](https://openrouter.ai/)

Once our accounts are set up, both of these will be used from inside VS Code, keeping our tools in one location so we don't need to leave our IDE to add agents to our workflow. We will also explore sandboxing as a way to secure agents, and look at what sandboxing controls VS Code provides.

## Learning Goals

- Enable GitHub Copilot and configure access in VS Code.
- Create an OpenRouter account and connect it to VS Code.
- Explain the purpose of a sandbox in an agentic coding environment and describe when different levels of sandboxing and agent access are useful. 
- Enable agent sandboxing in VS Code.

## Vocabulary and Synonyms

| Vocab | Definition | Synonyms | How to Use in a Sentence |
| --------- | --------- | -------- | --------- |
| Sandbox | An isolated execution environment that constrains what an agent can access or modify, limiting the blast radius of unintended actions. | Isolated environment | "Running the agent in a sandbox meant that even when it attempted to modify a file outside the project directory, the operation was blocked." |
| Container | A lightweight, isolated runtime environment that packages an application with its dependencies and gives it its own view of the filesystem, network, and processes, separate from the host system. | Docker container, isolated environment | "We mounted only the project directory into the container, so the agent had no visibility into the rest of the host filesystem." |

## Copilot Set Up

Github's Copilot is a paid service that is free for students for a limited time through the Github Student Developer Pack. When redeeming the Student Developer Pack for any of the other benefits it provides, we also gain access to Copilot, but we need to turn on the service. 

### Enabling Copilot

To enable Copilot access:

1. Check if your Student Developer Pack is currently active.
   - Look for a "PRO" status under "Highlights" in your profile on GitHub. In the upper-right corner of any page, click your profile photo, then click "Your Profile".  
   ![A GitHub profile page with the "Highlights" section circled](../working-w-copilot/assets/setting-up-copilot/github-profile-pro-location.png)  
   *Fig. "Highlights" section of a profile on GitHub*
   - If you have issues accessing or activating your Student Developer Pack, please file a ticket with [GitHub Education Support](https://support.github.com/contact/education). The linked support page also has a button to troubleshoot with GitHub's Virtual Agent in case it can provide immediate help. 

2. Go to the account settings screen by clicking your profile photo in the upper-right corner of any page, then click "Settings". Under the "Code, planning, and automation" section of the "Settings" sidebar, click "Copilot".  
   ![The GitHub "Settings" menu with the "Code, planning, and automation" section underlined and the "Copilot" option circled](../working-w-copilot/assets/setting-up-copilot/github-settings-menu.png)  
   *Fig. "Copilot" option in the "Settings" menu*

3. Click “Enable Copilot”. A message will appear that we have access to Copilot for free, along with a new button “Continue to get access to Github Copilot” that we should click.

4. We will be taken to a preferences screen where we can choose some options about how Copilot will use public code and snippets it generates. Leaving the options on can help improve the overall quality of Copilot, but turning them off won't impact your ability to get recommendations.
   - Choose what you are most comfortable with, these choices can be updated at any time by navigating to "Settings" > "Copilot".

### Copilot in VS Code

To add Copilot to VS Code, we need to install the Copilot extension:

1. Open VS Code, then click the Extensions tab.
 
2. In the Extensions tab, search for "GitHub Copilot"  
   ![The "Extensions" tab of VS Code with the search terms "GitHub Copilot" entered in the search bar. The results show "GitHub Copilot" as the first result, which is circled in red.](../working-w-copilot/assets/setting-up-copilot/vscode-search-copilot.png)  
   *Fig. "GitHub Copilot" in the VS Code Extension search results*

3. We can install GitHub Copilot using the "Install" button directly on the entry in the results list or click the "GitHub Copilot" result to go to their extension page and click "Install" from there.
   - You may see other Copilot extensions listed, only the "GitHub Copilot" extension is necessary for what we will cover. 

   ![The information page for the GitHub Copilot Extension](../working-w-copilot/assets/setting-up-copilot/vscode-copilot-extension-page.png)  
   *Fig. "GitHub Copilot"'s extension page in VS Code*

4. If you haven't previously authorized VS Code in your GitHub account, you will be asked to sign in to GitHub inside VS Code. If you've previously authorized Visual Studio Code in your GitHub account, GitHub Copilot should automatically be authorized.
   - In your browser, GitHub will request the necessary permissions for GitHub Copilot. To approve these permissions, click Authorize Visual Studio Code.
   - In Visual Studio Code, in the "Visual Studio Code" dialog box, click "Open" to confirm the authentication.

For instructions to set up the Copilot extension in VS Code written another way, check out GitHub's ["Getting started with GitHub Copilot"](https://docs.github.com/en/copilot/get-started/quickstart?tool=vscode) documentation.

As mentioned earlier, we will be using Copilot and OpenRouter exclusively from VS Code in this curriculum, but feel free to follow your curiosity and look up whether plugins are available for any other IDEs you are interested in trying!

## OpenRouter Set Up

To use OpenRouter, we will need to:
1. Create an account
2. Create an API Key
3. Add the API key to VS Code

### OpenRouter Account and API Key

1. Browse to [OpenRouter](https://openrouter.ai/) and click the "Sign Up button".

    ![OpenRouter homepage with sign up button highlighted](assets/set-up/openrouter_home.png)
    *Fig. OpenRouter homepage*

2. This will open a pop-up that lets you create an account using a username & password or select from other auth options. Choose the option you prefer; once your information is filled out and submitted, you should receive an email to confirm your account.

    ![OpenRouter "Create Account" pop up showing multiple modalities to choose](assets/set-up/openrouter_sign_up.png)   
    *Fig. OpenRouter "Create Account" UI*

3. After clicking the link in your email to confirm your account you may be brought either to a home screen or directly to the "Credits" billing page. You are required to add payment information to your account, however, you are not required to add any credits to OpenRouter to access the free models they host. If you are not immediately brought to payment information, it can be accessed from the menu at the top right of the page:
    ![OpenRouter's top right menu open with Credits section highlited](assets/set-up/openrouter_billing.png)
    *Fig. OpenRouter menu open with "Credits" section highlited ([Full Size Image](assets/set-up/openrouter_billing.png))*
    - There are tight request limits on free tier models (at the time of writing this, 50 requests per day if you have not added credits to your account). If you add $10 in credits to your account, OpenRouter increases the free tier request limit (up to 1,000 requests per day, at the time of writing this lesson). Check out OpenRouter's ["Limits"](https://openrouter.ai/docs/api/reference/limits) guide for updated information on current limitations.

4. Once billing information has been added, OpenRouter will generate a default API key for us that has no restrictions and display it for us to copy. We can use this key, but especially if you add credits and might experiment with paid models later, we recommend creating a new key. This is because there are guardrail options we can set when creating a key that limits how many credits the key can use before deactiviating to help control our spend. 
    - **Default Key**: If you'd like to use the default key, copy the value when it is shown, as the API key is only displayed once. We can always invalidate the current key and generate a new one if we forget to copy or lose it before adding it to VS Code.

    - **New API Key**: To create a new API key: 

        1. Click the top right menu and select "Workspaces" from the drop down. On the workspaces page there is a menu on the left of the page, click "API Keys". 
            ![Screenshot of the OpenRouter workspaces site showing the steps to navigate to the API Keys page](assets/set-up/openrouter_workspaces.png)
            *Fig. Navigation to the API Keys view in OpenRouter ([Full Size Image](assets/set-up/openrouter_workspaces.png))*
        
        2. On the "API Keys" page, click the "New Key" button towards the top right.
            ![OpenRouter's API Keys page with the "New Key" button highlighted](assets/set-up/openrouter_new_key_button.png)
            *Fig. Where to create a new key on OpenRouter's "API Keys" page ([Full Size Image](assets/set-up/openrouter_new_key_button.png))*
        
        3. In the pop up that displays, you are required to choose a name to help you distinguish between your keys. Nothing else is required to create a key, but we recommend looking through the options, since this is where we could add restrictions like a spending limit to the API key. 

            ![OpenRouter's UI for creating a new API key](assets/set-up/openrouter_new_key_ui.png)   
            *Fig. OpenRouter's form for creating a new API key*

        4. Once the key is created, it will only be displayed once. Copy the value and hold onto it securely since we'll need it in the next section!

### OpenRouter in VS Code

We need to add OpenRouter to VS Code so we can choose models they provide from inside our IDE. To do this, we need to add OpenRouter as a provider and supply our API key.

1. Inside of VS Code, use the shortcut "Shift" + "cmd" + "i" (`⌃⌘I`) to open the chat view. Click the model picker button at the bottom of the chat pane. This field will show the name of the currently selected model or "Auto" if the IDE is set to automatically pick from available models.
    ![VS Code chat pane with the model picker button highlighted](assets/set-up/vscode_chat_models_button.png)   
    *Fig. VS Code's model picker button in the chat UI*

2. In the pop up that displays, press the gear icon in the lower right corner to go to the model settings view.
    ![VS Code showing the model picker UI with the model settings icon highlighted](assets/set-up/vscode_model_settings_button.png)   
    *Fig. VS Code's model picker UI with the model settings icon highlighted*

3. From the model settings screen, press the "Add Models" button next to the search bar at the top of the frame, then select "OpenRouter" from the drop down that appears.
    ![VS Code's model settings screen showing the "Select a provider" drop down when adding a new model](assets/set-up/vscode_manage_models_view.png)   
    *Fig. VS Code's drop down for choosing a provider when adding new models ([Full Size Image](assets/set-up/vscode_manage_models_view.png))*

4. A prompt will appear at the top of the screen asking for a name to group the OpenRouter models under in the UI. "OpenRouter" will be entered in the field by default, press enter to accept this name.
    ![VS Code UI for creating the group name for a set of models](assets/set-up/vscode_name_provider_group.png)   
    *Fig. VS Code's text field for naming the model group ([Full Size Image](assets/set-up/vscode_name_provider_group.png))*

5. A new prompt will appear at the top of the screen asking for your API key. Paste in your OpenRouter API key and press enter.
    ![VS Code's UI for adding a model provider's API key](assets/set-up/vscode_add_api_key.png)   
    *Fig. VS Code's text field for adding a provider's API key ([Full Size Image](assets/set-up/vscode_add_api_key.png))*

6. After a moment, the model settings ui should refresh and we should be able to see a new section and many more models that we now have access to. 
    - If at any point we need to update our API key or want to remove OpenRouter from VS Code, we can press the settings icon on the group name for options. 

    ![VS Code's model settings screen updated to show OpenRouter and its available models](assets/set-up/vscode_new_models_showing.png)   
    *Fig. VS Code's model settings screen showing OpenRouter models and settings to update or remove the model provider ([Full Size Image](assets/set-up/vscode_new_models_showing.png))*

### Choosing Models with OpenRouter

We now have access to multiple models and model providers! To see what models are curently free on OpenRouter, check out their ["Models" page](https://openrouter.ai/models) and use the filter options to select "Text" input and "Free" pricing. 
    ![OpenRouter's "models" page with filters for input type and cost highlighted](assets/set-up/openrouter_models_page_filtered.png)   
    *Fig. OpenRouter's "models" page with filters for input type and cost highlighted ([Full Size Image](assets/set-up/openrouter_models_page_filtered.png))*

When you see one that you like, we can search for that name under the model picker in VS Code!
    ![VS Code's model picker in the chat pane with the search bar highlighted](assets/set-up/vscode_model_picker_search_box.png)   
    *Fig. We can search models in VS Code using the search bar at the top of the model picker UI*

## Sandboxing and Scoping Access

Before we dive into using the models we just made accessible, let's talk a little about security. We've established that LLMs and agents built on them operate on statistical text prediction, not by reasoning about consequences. An agent's context might deem a particular file write statistically likely (such as editing an existing file, creating a new configuration file, or overwriting a file completely) even if it is not what is actually correct or secure. The agent might also produce a command that seems like a natural next step, but might not actually be safe to run.

Sandboxing is how we enforce constraints that prompts alone cannot reliably maintain. A **sandbox** is an isolated environment that constrains what an agent can reach. By scoping the access that AI agents have, we limit the potential blast radius of unexpected agent actions. 

### Why We Sandbox Agents

Without sandboxing, an agent with file system access can write to any location our user account has permission to write to. An agent with shell access can run any command available in that shell. This is rarely what we want! The most common problems sandboxes protect against are:

- **Filesystem scope creep**: Without boundaries, an agent with file write access can write anywhere our user account has permission to write, including dotfiles, other project directories, or SSH keys.
    - Imagine asking an agent to clean up temporary files in your project. The agent interprets this broadly and starts deleting anything that looks temporary  like cache files in a sibling project directory that happened to be within reach. Without a sandbox boundary, 'clean up temporary files' turned into 'delete things outside the intended project'.

- **Credential exposure**: Local environments often have API keys or cloud credentials sitting in environment variables or config files. An agent with unrestricted read access can pull these into its output or logs without any malicious intent behind it.
    - Imagine asking an agent to debug why an API call is failing and it prints out the full request, including a header with your API key, into a log file that later gets uploaded to a public GitHub repo for troubleshooting. The key is now public, and anyone can use it until it's revoked.

### The Sandboxing Spectrum

Sandboxing isn't one setting we turn on, it is a suite of tools that work on different levels of our set up. Each level of sandboxing trades more setup effort for a stronger guarantee about what is contained.

| Level | What it constrains | Setup effort |
|---|---|---|
| **Application permissions** | Categories of action: file reads, file writes, running commands, network access. Usually configurable as allow, deny, or ask. | Built into most agentic tools already, little to no setup |
| **OS-level isolation** | Specific file paths or system calls a process can use, enforced by the operating system itself (for example, macOS's `Seatbelt`). | Low to moderate, some tools ship this ready to enable |
| **Container isolation** | Everything outside an explicitly mounted directory. The agent's filesystem, network, and process view are fully separated from the host machine. | Highest, typically requires Docker or a similar runtime tool |

**Application-level permissions** are the coarsest tool: they draw lines around categories of action, not specific paths, so "allow file writes" means the agent can write anywhere our account can. 

**OS-level isolation** tightens this by enforcing specific file path or system-call boundaries below the application, so even a misbehaving tool can't bypass the restriction. 

**Container isolation** is the strongest option, fully separating the agent's environment from the host, at the cost of needing Docker and some setup around what gets mounted into the container (what folder or volumes are visible inside the container).

For most development work, the choice comes down to risk profile and workflow needs:

| Level | Good for | Limitations |
|---|---|---|
| Application permissions | Quick tasks, supervised sessions, low-sensitivity projects | Category-level only, below-application bypasses aren't blocked meaning the permissions control which apps get access, but not what an agent does once inside. |
| OS tools (Seatbelt, seccomp, firejail) | Single-machine setups where containers add too much overhead | Setup complexity, misconfiguration risk |
| Container isolation | Autonomous runs, scripts from external sources, sensitive environments | Docker required, initial setup overhead including determining what should be accessible from the conatiner |

Starting with whatever sandbox our tool provides by default is a reasonable place to begin. As tasks run longer, touch more of the filesystem, or operate near sensitive credentials, moving up this spectrum becomes worth the added setup.

### A Practical Starting Point in VS Code

VS Code gives us tools at the first two levels without any extra setup, which makes it a good place to start before we consider anything container-based.

**Permission levels** control how much a session runs without pausing for our input. The default, "Default Approvals," shows a confirmation dialog before tool calls that modify files, run commands, or reach external resources. "Bypass Approvals" and "Autopilot" skip that dialog entirely. 
- For early agentic coding work, staying on Default Approvals gives us a chance to catch a command before it runs rather than after.

**Terminal auto-approve rules** let us pre-approve specific commands or patterns so we aren't confirming the same safe command repeatedly. This reduces friction, but it's worth knowing the limitation going in: VS Code parses commands using best-effort grammars, and shell tricks like quote concatenation can slip past the rules undetected. Auto-approve rules cut down on interruptions, they don't enforce a boundary the way OS-level isolation does.
- We should be very careful about what commands we auto-approve when we're getting started. As examples, a command that creates or enables a `venv` is pretty safe, but allowing new dependencies to be installed without your awareness could both bloat the project and bring in packages with security vulnerabilities.

**Agent sandboxing** is VS Code's OS-level option. It's the setting `chat.agent.sandbox.enabled` on macOS and Linux, or `chat.agent.sandbox.enabledWindows` on Windows. We can find it by searching "chat.agent.sandbox.enabled" inside VS Code's settings: 

![VS Code's settings UI open showing the checkbox to turn on agent sandboxing](assets/set-up/vscode_enable_sandbox.png)
*Fig. The Sandbox setting enabled in VS Code's UI ([Full Size Image](assets/set-up/vscode_enable_sandbox.png))*

We recommend that folks turn this setting on to give some security and start getting some exposure to how we can scope access for our tools. You can access your settings with the shortcut `cmd` + `,`. With it enabled, terminal commands run inside a boundary enforced by the operating system:

- **File reads** are limited to our workspace folders and a few paths tools like `git` or `npm` need to function. Our home directory is off-limits by default, which keeps things like SSH keys and shell config out of reach.
- **File writes** are limited to the current working directory and its subdirectories.
- **Network access** is blocked by default, though we can allow specific domains if a command needs to reach one.

Because the boundary is enforced by the OS, commands that stay inside it run without a confirmation prompt. Anything that needs more than the sandbox allows surfaces a prompt asking whether to run it outside the sandbox instead. The isolation mechanism underneath differs by platform, but VS Code manages those details once the setting is turned on.

For our first agentic coding sessions, a reasonable combination is: 
1. keep the permission level on Default Approvals
2. turn on agent sandboxing

This pairs a human checkpoint before risky actions with an OS-enforced boundary in case something proceeds that we didn't anticipate, at the cost of a single settings change. As we mentioned earlier in the lesson, over time our tasks may grow more autonomous or start running unattended for longer stretches. At that point container isolation becomes worth the added setup, but VS Code's built-in sandboxing is enough to get us started.

## Next Steps & Notes

Before moving on to the next lesson, we recommend opening up a project that you are familiar with (any project from Unit 1 would work well) in VS Code. Our goal is for students to keep a coding project up while reading through the lessons and pause to try things out as they go.
- Any project can be used to try things out, but using a familiar project means that we can focus on the new topics and what is or is not working well with the agentic tools without also ramping up on a new codebase.

### VS Code Shortcuts

We won't be covering VS Code's UI shortcuts in depth, but there are a number of handy commands for actions like opening the AI chat view and starting a new agent session. 
- Check out VS Code's [AI Features Cheat Sheet](https://code.visualstudio.com/docs/agents/reference/ai-features-cheat-sheet) for a quick guide on commands
- Feel free to explore VS Code's documentation on [Coding with Agents](https://code.visualstudio.com/docs/agents/overview) if there are topics you want to dive deeper on!

### Other LLM Providers

We will only be working with these two model providers in the curriculum, but if you find other providers of free models, the same steps we used for OpenRouter can usually be followed to make their models available inside VS Code. 
- This can be a great way to get more agent usage if Copilot and OpenRouter's free tiers have been exhausted until the next refresh period. 
- Feel free to experiment and share useful tools and models with the class on Slack!

Students that already have or choose to later procure their own subscriptions to other LLM providers are welcome to use those, but instructor support for how to integrate those into the Ada toolset may be limited. 

## Summary

In this lesson we set up the tools we'll use for agentic coding and looked at how to keep those tools within safe boundaries. 

We walked through activating GitHub Copilot with the Student Developer Pack and installing its VS Code extension, then created an OpenRouter account, generated an API key, and connected it to VS Code as an additional model provider. 

On the security side, we looked at why sandboxing matters. 
- Since agents generate output based on statistical likelihood rather than an assessment of whether an action is safe, external constraints are what keep file access and command execution within safe bounds. 
- We compared three levels of sandboxing (application permissions, OS-level isolation, and full container isolation), each trading more setup for a stronger guarantee. 
- Finally, we ended with a practical starting point in VS Code: keeping approvals on their default setting while turning on VS Code's built-in agent sandboxing to restrict file and network access at the OS level.

## Check for Understanding

<!-- prettier-ignore-start -->
### !challenge
* type: checkbox
* id: hF0EIqaqXEuiy67YxscCNvMUyq
* title: Setting up Our Workspace
##### !question

Rather than using the default API key OpenRouter generates for us, we decide to create a new key instead. Which of the following are reasons this can be a good idea?

##### !end-question
##### !options

* A new key lets us set guardrails, such as a spending limit that deactivates the key before it uses up more credits than we intend.
* We can give a new key a name, which makes it easier to track usage if we end up creating multiple keys for different purposes.
* Creating a new key is the only way to connect OpenRouter to VS Code; the default key cannot be used at all.
* Creating a new key removes the requirement to add payment information to our OpenRouter account.

##### !end-options
##### !answer

* A new key lets us set guardrails, such as a spending limit that deactivates the key before it uses up more credits than we intend.
* We can give a new key a name, which makes it easier to track usage if we end up creating multiple keys for different purposes.

##### !end-answer
##### !explanation

The default key OpenRouter generates has no restrictions, so it works for connecting to VS Code, but it doesn't offer any spending controls. Creating a new key lets us name it and optionally add guardrails, like a spending cap, to help manage costs if we later add credits or try paid models. Payment information is required on our OpenRouter account regardless of which key we use.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: BbnvAEMPaZ9ZeHN0WBgAbpYeOG
* title: Setting up Our Workspace
##### !question

An agent is given unrestricted file system and shell access, and later writes to a file well outside the project directory. The agent wasn't instructed to do this and had no malicious goal. Which explanation best fits why this happened?

##### !end-question
##### !options

* The agent weighed the risk of the action and incorrectly concluded it was safe.
* The agent produced whatever action was statistically likely given its context, regardless of whether that action was actually safe to run.
* This can only happen if the agent was deliberately prompted to act outside the project directory.
* Agents are physically incapable of accessing paths outside the project directory unless a sandbox is disabled.

##### !end-options
##### !answer

* The agent produced whatever action was statistically likely given its context, regardless of whether that action was actually safe to run.

##### !end-answer
##### !explanation

Agents don't evaluate whether an action is safe before producing it. They generate output based on statistical text prediction, so if a file write or command is statistically likely given the surrounding context, the agent will produce it whether or not it's actually safe. This is exactly why external constraints like sandboxing matter: they enforce boundaries that prompting alone can't reliably guarantee.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
### !challenge
* type: multiple-choice
* id: UDIHTQVx05hLJxTLYU9PvZouw7
* title: Setting up Our Workspace
##### !question

We're about to let an agent run an unattended, multi-step script from an external source against a sensitive project, and we want the strongest available guarantee that it can't touch anything outside a specific folder. We're willing to accept extra setup time in exchange for that guarantee. Which level of sandboxing best fits this situation?

##### !end-question
##### !options

* Application-level permissions, since they can be set to deny file writes and command execution entirely.
* OS-level isolation, since tools like Mac OS's `Seatbelt` require no additional setup to use.
* Container isolation, since it fully separates the agent's filesystem, network, and process view from the host machine.
* No sandboxing is needed as long as we only run scripts we've read in advance.

##### !end-options
##### !answer

* Container isolation, since it fully separates the agent's filesystem, network, and process view from the host machine.

##### !end-answer
##### !explanation

Container isolation is the strongest point on the sandboxing spectrum: everything outside an explicitly mounted directory is fully separated from the host machine, which fits a high-risk, unattended scenario. It also requires the most setup, typically a runtime like Docker and a decision about what to mount into the container, which is the tradeoff we accept for that stronger guarantee. Application-level permissions only draw lines around categories of action, and OS-level isolation, while lower effort, is a lighter guarantee than full container separation.

##### !end-explanation
### !end-challenge
<!-- prettier-ignore-end -->