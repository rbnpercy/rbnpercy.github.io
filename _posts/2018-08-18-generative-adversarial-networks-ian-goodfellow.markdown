---
title: "Generative Adversarial Networks - Ian Goodfellow"
description: ""
date: 2018-08-18 09:30
tags:
- generative-adversarial-networks
- adversarial-machine-learning
- generative-modelling
- ian-goodfellow
- unsupervised-machine-learning
- machine-learning
- artificial-intelligence
---

**TL;DR**  Goodfellow mainstreamed the idea of GANs a few years ago, initially whilst trying to problem solve as a favour for a friend.  There's nothing new about training machine learning models and algorithms - Arthur Samuel's checkers agent back in 1959 was the original example of reinforcement learning.  However, with the introduction of Generative Adversarial Networks, Goodfellow has pioneered another class of super exciting algorithms for Semi / Unsupervised ML.


Generative Adversarial Networks - Ian Goodfellow
==============================

South Park Commons is a community that helps entrepreneurs and technologists freely learn and start ambitious projects. We bring together talented people to share ideas, explore directions, and realize opportunities that occur in an environment that helps people take risks. People don’t come to the commons to play it safe—we’re here to dive off the deep end.

As part of that journey, we bring in some of the best and brightest minds working on the cutting edge of tech to speak to our community. With our Artificial Intelligence Series, we’re sharing lectures on Machine Learning and other hot AI topics. Follow us here to get updated when new content is released.


[**Slides for this talk can be found here.**](http://www.iangoodfellow.com/slides/2018-05-24.pdf)


Ian Goodfellow obtained his B.S. and M.S. in computer science from Stanford University and his Ph.D. in machine learning from the Université de Montréal, under the supervision of Yoshua Bengio and Aaron Courville. After graduation, Goodfellow joined Google as part of the Google Brain research team. Later he left Google to join the newly founded OpenAI institute. He returned to Google Research in March 2017.

Goodfellow is best known for inventing generative adversarial networks, an approach to machine learning. He is also the lead author of the textbook Deep Learning. In 2017, Goodfellow was named in MIT Technology Review's 35 Innovators Under 35.

<br/>
## Adversarial Machine Learning
<br/>

> "I thought Generative Adversarial Networks (GANs) was going to be a breeze!  Four years later, I'm still working on it!"

Goodfellow mainstreamed the idea of GANs a few years ago, initially whilst trying to problem solve as a favour for a friend.  There's nothing new about training machine learning models and algorithms - Arthur Samuel's checkers agent back in 1959 was the original example of reinforcement learning.  However, with the introduction of Generative Adversarial Networks, Goodfellow has pioneered another class of super exciting algorithms for Semi / Unsupervised ML.

Machine Learning really only reached acceptable success rates ~5 years ago, and although it has advanced leaps and bounds in the last few years, most AI researchers will agree - it still has a **long** way to go.  In traditional ML, we have one player and one cost.  The player being the *model* or *classifier*, and the cost being the *log likelihood* or *reward function*  For years, researchers have focused on the optimisation of the minimum cost outputting the maximum reward.

In Adversarial ML, a multi-player and multi-cost paradigm comes into fruition.  With a multi-player paradigm comes conflict, but in this case the conflict is a good thing.  A real world example of these conflicts may include building security systems around ML, such as spam filters, and a bad actor trying to beat these spam filters.

GANs run on the Adversarial Nets Framework, consisting of a *Generator* and a *Discriminator*.  The *Generator Network* takes in random noise, and transforms that noise into a sample from the highly trainable model.

The *Discriminator* acts as a binary classifier, evaluating the quality of an image, and assigning a floating point integer to the input between 0 and 1, specifying how likely it is to be real or fake.  When data from the *Generator* is fed to the *Discriminator* a value closer to 0 should be achieved, demonstrating a generated / fake sample.  The *Discriminator* is trained on real data, while simultaneously the *Generator* is constantly feeding and trying to trick the *Discriminator* with fake data.

Generative models are still improving, day by day.  Previously, convoluional models have been the popular form of generative modelling, but convolutional functions have their downsides, such as copying the same kernel across images, leading to counting errors.  Convolutional models are great for checking the existence (or lack thereof) of items in images, but the lack of counting skill leads to errors such as dogs generated with three heads etc.  In GANs, the self-attention functionality takes care of these counting errors, leading to a demonstrable leap forward in quality.  With this, GANs are already appearing in the real world.

> "Personalised **GAN**ufacturing"...

...Is a term Goodfellow uses in his talk to describe the revolutionary way consumer product industries can be transformed.  Already, there is a Dental company called *Glidewell Dental* utilising GANs to produce dental crowns, 3D printed, and personalised for consumers.

<br/>
## Privacy and Security
<br/>

> "We'll never have an un-foolable algorithm.  It's *really* easy to fool ML models."

One of the big explosions in research topics in recent times, is security and privacy issues in AI.

Bad agents can use Adversarial attacks to reprogramme neural net functionality by introducing peturbations (adversarial data), in the form of transformations to input data.

![]/assets/img/(panda-to-gibbon-gan-security.png) 

In the above image, an attacker has taken the image of the panda, and added a carefully calculated vector to the image, scaled down enough so *we* can't actually see the change to the image on the right.  However, the image has gone from being identified at **70%** as a Panda, to **99.9%** as a Gibbon.  In this attack, not only has the bad actor tricked the model about the image content, the model is also now even more confident about the image contents.

Utilising GANs, a bad actor could also hijack computing resources to complete tasks such as forcing a classifier in a cloud-hosted image service to solve captchas, or even to mine for cryptocurrencies.

That being said, adversarial models have a lower error rate than previotus ML models, and also change more frequently than other models, making them better for security when used for good.  By generating adversarial data meant to fool ML systems, GANs are a great way of improving security, learning from erronious results.

> "In GANs, unlike many technologies, Privacy is actually in favour of the defender, over the attacker."

In his talk, Goodfellow mentions ***Differential Privacy***.  The term was coined by *Cynthia Dwork* back in 2005, as a way of accessing the database without revealing private data.  The term differential in this case, relates to taking two datasets, one which includes specific records, and one which does not, and the difference between the two.  Relating to GANs, Goodfellow takes the example of creating a randomised process that can learn parameters in a way that the data insights output is the same, regardless of the difference in datasets, therefore preventing privacy leaks.  Goodfellow also touches on the uses of *Teacher Networks* trained on public datasets, utilising noisy aggregation of data. With these techniques, Goodfellow shows that GANs can *far* reduce the probability of a bad actor being able to reverse engineer parameters and access private data.

<br/>
## Extreme Reliability, Efficiency and Fairness

One of the most controversial topics of machine learning is *reliability*.  ML is being implemented into our everyday lives, and more often into systems that need ***extreme** reliability*, such as air traffic control systems, surgery robots, medical diagnoses and autonomous vehicles.

Although not mentioned in Goodfellow's talk, I must add that GANs are already being proven in these extreme reliability scenarios, such as a transportation system for autonomous vehicles that was recently built based on deep reinforcement learning, benchmarking collision avoidance under worst-case scenarios.  These scenarios were created by an adversarial agent trained to drive the system into unsafe situations.

> "Commercially, one of the hottest topics is efficiency."

As Goodfellow states, *efficiency* is one of the most researched topics in ML.  If one can assemble a large dataset, one can train a highly effective ML algorithm.  Or so the theory goes.  Goodfellow touches on label efficiency by means of Virtual Adversarial Training.  While adversarial training requires access to data labels to perform perturbations, virtual Adversarial Training requires no labels and is therefore suitable for semi-supervised learning. Virtual adversarial training effectively seeks to make the model robust to perturbations where it is most sensitive, and has so far achieved good efficiency results in cases such as text classification datasets.

> "We need to go out of our way to ensure ML algorithms perform better than people."

In the above quote, Goodfellow is actually refering to ***fairness*** in Machine Learning, as opposed to *efficiency*.  There are a lot of algorithms that affect people's lives, such as hiring algorithms or loan decisions, and we need to ensure that these systems don't include human biases.  In ML, there is a phenomenon called **bias amplification**, in which algorithms pick up on the human trainers if they include any kind of prejudice at all.

It's not enough to simply remove these sensitive or protected variables from our data, but by assigning **S** a sensitive variable (be it gender, race or income), with Adversarial process' we can make it impossible for the *second player* to guess that sensitive variable, and therefore prevent bias against it.  Using GANs in this context, a final decision would never be made based around any human bias.


<br/>
## Where Are We Now?

Adversarial ML is yet another tool that proves both the brilliance &amp;  effectiveness of ML, ***and*** the idiosyncrasies &amp; inefficiencies. The fact is that as of right now, ML models are often very easy to fool, and even linear models work in counter-intuative ways.  Much of the available *Adversarial ML* literature agrees with this statement, although many ML researchers would disagree with me here.

As far as Adversarial ML goes, it is another impressive step forward in Machine Learning, and has a promising future, backed up by a team of great minds in an ideal position to push such ideas forward.

As a final note, as in the talk video, I'd just like to end with an image generator using GANs:

![](/assets/img/gan-cat.jpg) 

Generated from a dataset of cat pictures from the web, GANs have created a fantastically realistic image here.  Worth noting is the nonsensical text appearing on the image, generated from the amount of cat images containing meme text.  Overall, as Goodfellow demonstrates in his talk, I believe GANs have a **very** bright future.

[ - ***@Rbin***](http://robin.percy.pw)