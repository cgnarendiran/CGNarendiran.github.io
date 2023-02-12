---
layout: post
title:  "ChatGPT - The Conversational Wizard"
date:   2023-02-10
image:  images/blog12/cover.png
tags:  chat gpt nlp llm language models gpt3 bert openai microsoft
---
*On the cover: ChatGPT's response to the prompt "What is the meaning of life?"*

As we are at the cusp of an [AI war](https://www.theverge.com/2023/2/7/23587767/microsoft-google-open-ai-battle-search-bing) between Google and Microsoft, I think it's high time we understand what's going on under these conversational AI models. If you have tried [ChatGPT](https://chat.openai.com/), you might have noticed its ability to keep track of context in conversation, calculate math and even code! Of course, it gets it wrong sometimes because, it is not trained to be a subject matter expert, but rather a language model expert. But there's no doubt that it has revolutionized the way we interact with an AI chatbot. How did they make it work? Let's break it down!

You're probably aware that the underlying model of ChatGPT is from the improved [GPT-3.5 series](https://platform.openai.com/docs/model-index-for-researchers) of models by [OpenAI](https://openai.com/) and the architecture is a [Transformer](https://jalammar.github.io/illustrated-transformer/). But [GPT-3](https://arxiv.org/abs/2005.14165) which made a big wave, was released in May 2020 and has been around for a while. How is ChatGPT different from regular GPT-3 and what makes it special? Let's look at the responses for the same prompt on different models:

<span style="color:#8b4950; font-family: Monospace">**Prompt** [model: `myself-lol`]</span>: Good evening! How are you?

<span style="color:#8b4950; font-family: Monospace">**GPT-3** [model: `davinci`]</span>: “I’m fine, thank you.”
“I’m sorry to bother you, but I’m looking for a friend of mine.”
“Oh?”
“Yes. His name is John.”
“John?”
“Yes. John.”
“John who?”
“I don’t know"

<span style="color:#8b4950; font-family: Monospace">**InstructGPT** [model: `text-davinci-003`]</span>:
Good evening! I'm doing well, thank you. How about you?

<span style="color:#8b4950; font-family: Monospace">**ChatGPT** [model: not-released]</span>:
Good evening! I am an AI language model, so I don't have feelings. But thank you for asking. How can I assist you today?

We can see that GPT-3 is generating some script encountered in its training corpus and is not very helpful to us. Compare this to InstructGPT whose response looks conversational. And finally, we have ChatGPT whose response is not only conversational but also mentions that it doesn't have feelings as a language model and plays the role of an assistant. This difference is because GPT-3 is only trained to complete the next word in a sentence. Whereas InstructGPT and ChatGPT not only use GPT-3's capabilities to comprehend language, but are also refined with conversational text and human feedback. Essentially three things are incorporated in ChatGPT:
1. Generative Pre-training (GP)
2. Supervised Fine-tuning (SFT)
3. Reinforcement Learning from Human Feedback (RLHF)

Generative Pre-training (GP) is where the model is trained to predict the next word $x_{t+1}$ given the context of the current sentence $p_\theta(x_{t+1}\vert x_0, x_1, x_2, ... x_t)$. The model learns the nuances and structure of language. GPT-3 from OpenAI or [PaLM](https://arxiv.org/abs/2204.02311) from Google are trained this way. Since ChatGPT is based on GPT-3, it basically is a large language model. But as we saw earlier, this is not enough. We want the model to follow our instructions and not just make generic completions. In GP, the end task (following instructions) is not the same as the training task (generic completion), so there is an [alignment problem](https://openai.com/alignment/). 

Now comes Supervised Fine Tuning (SFT) to the rescue. This is where the pre-trained model is fine-tuned with conversational data. OpenAI hired 40 independent contractors (from UpWork and ScaleAI) and asked them to come up with coversational dialogue on variety of use cases that include Generation, Open QA, Chat, Summarization, Classification, Extraction and so on. OpenAI utilized the user-prompts they received on their [playground](https://platform.openai.com/playground) on an earlier version of this model.

![alt](/images/blog12/training_sft.png){: .center-image }
*Figure 1: Supervised Fine-tuning (SFT) process*

This is fine, but how is the model is able to remember what happened so far in the conversation and respond accordingly? Cosider a long dialogue with query-answer pairs given by the labelers. When feeding this into the model, you not only give the current query, but all the history of query-answer pairs before it. This way the model learns to keep track of the context in dialogue. In a dialogue with $K$ set of query-answer pairs, you'd get $K-1$ set of consolidated query-answer pairs.

![alt](/images/blog12/consolidated_query.png){: .center-image }
*Figure 1: Consolidated query for incorporating context*

Now, one would think that SFT should be enough, but what is the need for RLHF? Let's say that you ask this SFT model a question that it has not encountered in its training set. If the model is generic enough, it can handle it, but if the question falls way off the training data distribution, then it suddenly starts failing. Imagine asking a simple dog/cat classifier model to classify a picture of an elephant.

Let's think of this from a Reinforcement Learning (RL) framework. The current observation in this setting is the sentence input, the agent is our model and it takes an action which is the next word in the sentence. Since we have both the input sentence and output predictions, we essentially have the state-action pairs as our data. So we can learn this mapping using a function approximator such as neural network **without obtaining the rewards** or designing a reward function. This type of fundamental Supervised learning is also known as [Imitation learning](https://smartlabai.medium.com/a-brief-overview-of-imitation-learning-8a8a75c44a9c) in the RL context.

Imitation Learning is a huge topic in itself. The vanilla Imitation Learning is also known as Behavior Cloning. But just as I mentioned before, if the agent makes a mistake (which it will) and gets into a state (say elephant) that it did not encounter in the expert's demonstration (cat or dog), it starts generating nonsense and making cocky assertions.

One of the ways to mitigate this is Inverse Reinforcement Learning (IRL) where the intent of the expert is captured by learning a Rewards Model. Now the Rewards model can be trained through the demonstration data, but it is tedious to give demonstrations. Will you rather rate/rank the agent's responses or write and show the correct response? The former is much easier, but difficult for the model to learn from this sparse information. The authors from OpenAI and Deepmind took this challenge (yes they came together!) and revived the [RLHF idea](https://arxiv.org/abs/1706.03741).

![alt](/images/blog12/rewards_model.png){: .center-image }
*Figure 8: Rewards model training from the comparison of responses by a human*

This is where RLHF comes into picture for ChatGPT. They generated multiple responses from ChatGPT (say $A, B, C, D$) and asked the human labelers to rank them (say the ranking was $B>C>D>A$). Among this ranking, we choose two at a time as input for the model to compare. For $K$ responses, you'd get ${K \choose 2}$ inputs to the model. For InstructGPT, $K$ was anywhere between 4 and 9 responses. The model will yield a reward for both the responses and then we use cross-entropy loss to maximize likelihood of the chosen response:

$$
L = - \sum_{A,B \in \mathcal{SFT}} (\mu(A) \log(p_\phi(A)) + \mu(B) \log(p_\phi(B)))
$$

where $\mu=1$ for the preferred response and $\mu=0$ for the rejected response. And $A,B \in \mathcal{SFT}$ are the responses that the SFT model has generated. The Rewards Model trained by OpenAI was initialized from the SFT model, however they used the smaller version of SFT with 6B parameters instead of the 175B. This is because the Rewards model will be used in the next step as a value function (critic in PPO) and hence stability and quicker computation are important.

![alt](/images/blog12/training_ppo.png){: .center-image }
*Figure 1: Final model training with PPO and feedback from RM*

Once this is done, we can use a standard RL algorithm such as [Proximal Policy Optimization (PPO)](https://arxiv.org/abs/1707.06347) to train an agent/model to give us the intended responses. In this case, they initilized the weights of the PPO agent from the SFT model and generated responses. These responses are passed through the Rewards model to get a reward. This reward can then be used to train the model's policy. Algorithms such as PPO are designed to encourage behavior that maximizes the cumulative reward. Hence the model is incentivized to output sentences that align with human preferences. This is how ChatGPT is trained and this is why you get such good responses from ChatGPT.

There are considerations apart from helpfulness of the model such as truthfulness, harmlessness, toxicity and misinformation. These aspects are fed implicitly into the model through labeler's demonstrations. However, this also means that the model is only as good as the labelers who gave demonstrations and preferences. As Sam Altman said in his tweet, "It's a mistake to be relying on it (ChatGPT) for anything important right now. It's a preview of progress; we have lots of work to do on robustness and truthfulness".

However, more powerful and useful models are on the way. As of writing this blog, Microsoft has announced the [integration of ChatGPT](https://blogs.microsoft.com/blog/2023/02/07/reinventing-search-with-a-new-ai-powered-microsoft-bing-and-edge-your-copilot-for-the-web/) (advanced version) into Bing search while Google has introduced [Bard](https://blog.google/technology/ai/bard-google-ai-search-updates/) which is based on their language model called [LaMDA](https://blog.google/technology/ai/lamda/). We will soon be taking these technologies for granted, just like we did Google Search 20 years ago. There will be a fundamental change in the way that information is searched and utilized. We are in the midst of an AI revolution and the best thing is that we are all a part of it.

That's it! This is the essence of ChatGPT and now you know :)