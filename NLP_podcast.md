## Topic Modeling from Podcasts with NLP
<br>

### Introduction  

Podcasts are an incredibly rich source of information. With millions of different shows and countless hours of content, the abundance of information in this format is staggering. However, extracting and searching for information within audio is inefficient and relies on the level of annotation from the producer. With the addition of audio transcription, podcasts get a lot more accessible in terms of search and large-scale analysis of content. Here, I use natural language processing (NLP) and topic modeling of podcast audio transcripts to characterize a set of podcast episodes and identify changes in topics throughout episodes.

For this project I chose to explore topics within the *Ask a Cycling Coach Podcast - Presented by TrainerRoa‪d‬* (hereafter referred to as the “TrainerRoad” podcast). This show is a great case study for topic modeling and extracting specific information. The content covers a wide range of topics related to cycling, so in addition to broad themes, the model can be tuned to capture more ancillary topics. The format mostly centers around Q&A segments or interviews focused on specific topics, but portions of episodes have more casual conversation, so we can see how well topics are identified in different formats. There is also a lot of content available. At the time of this project, there were 141 episodes with time-stamped transcripts with a median audio length of 115 minutes. The episodes were released over nearly 3 years, so we can see how content changes over time.‬‬ 

### Methods  

#### Collecting transcripts and metadata  

To get data from a set of episodes, I started with a YouTube playlist that includes all episodes from November 15, 2017 to present. Although this is not the full set of published shows, most of these have transcripts available. I used Selenium and Beautiful Soup to collect URLs from that playlist and extracted YouTube video IDs from URLs. With the YouTube Transcript API I captured time-stamped transcripts associated with each video ID. I also collected some metadata by navigating to each episode YouTube page and scraping upload date, likes, dislikes, views, and episode titles using Beautiful Soup.

#### Text processing and topic modeling

The YouTube transcripts for these shows were time-stamped, which allowed me to divide each episode into separate documents. I binned text into 5-minute chunks, so a 120-minute episode contains 24 documents. I tokenized the text with spaCy using the English core module, and simplified the corpus by lemmatizing nouns and adjectives to extract base forms of words. From these processed text strings, I created a vectorized matrix of token counts using the scikit-learn CountVectorizer with a maximum document frequency of 0.3 to include only the most document-specific words. To identify document topics from the token counts, I fit a Latent Dirichlet Allocation model. With some optimization I settled on a 16-component model to balance topic specificity and similarity.

### Results

#### Model optimization allows identification of specific and useful topics

Determining an optimal number of topics for the model is somewhat subjective and context dependent. For this analysis I wanted to be able to distinguish between somewhat similar topics, such as nutrition and heat & hydration (similar because both these might include words like water, sodium, or intake…), and at the same time avoid generating too many separate, and highly specific, topics (for example, separate topics for on-the-bike nutrition versus off-the-bike nutrition). Through some trial and error, optimizing the number of topics and max document frequency, and aggressive preprocessing/ filtering for only nouns, adjectives, and lemmatization, I was able to generate 16 topics that are informative about the general topic of conversation, but not overly specific. Here is an example of some of these topics and top words associated with each.

<img src="images/NLP_podcast/Topics.png?raw=true"/>

#### Topic frequency changes between episodes

To compare how topics change between episodes, I looked at the average frequency across documents within each episode. Most episodes are a mix of topics, and there are many that are frequently discussed on the show. Talk about training plans, workout structure, and MTB racing is very common and occurs in nearly every episode. However, specific topics stand out in some cases. In two episodes featuring a guest from Precision Hydration® who specializes in athlete hydration, nearly the entire episode is devoted to that topic. While it’s not a far stretch to guess that those episodes would be focused on the heat & hydration topic, it’s a good validation that the model is picking up on the signal. What isn’t as obvious without this type of analysis though, is that by plotting topic frequency through time we can see some interesting trends. In the case of road racing, there is some periodicity to how much this topic is discussed. The road race season begins in early spring, and we can see an increase in how much in road race topics around the same time each year. For this podcast, many episodes feature discussion of current experiences of the hosts (i.e., talk about recent and upcoming races/ events), which carry from week-to-week. So, for a listener who plans to listen to several episodes and is looking for discussion around a particular topic, a time-series view like this can help identify periods of time when shows are focused around a specific topic or series of events.

<img src="images/NLP_podcast/Periodic.png?raw=true"/>

#### Topic modeling identifies changes in topic throughout the course of an episode

Some of the episodes include chapter annotations on YouTube that summarize discussion at specific time points. To look at how my topic model compares to these annotations, I looked at modeled topic distributions within 5-minute documents through the course of two episodes. Of course, the actual podcast discussion doesn’t change on 5-minute intervals, nonetheless, we can still see correlation between annotations and topic distributions. In episode 281 for example, the annotation about fixing low back pain coincides with high representation of the physiology topic. We also see strength and riding – technical topics come up, which makes sense that these topics are related to fixing low back pain. While these topic labels are far from perfect, they give a good representation of topic continuity through time, and how any discussion is always a mix of multiple topics.

<img src="images/NLP_podcast/Episodes.png?raw=true"/>

#### Comparing topics between athlete interview episodes highlights common structural elements

While most TrainerRoad episodes are largely Q&A format, there are some episodes that feature interviews with guests. After looking at how topics change through the course of a normal Q&A episode, I was curious what topic distribution would look like in interview episodes. As an example, I looked at episodes featuring interviews with two high profile cyclists, Justin Williams, a road cyclist, and Kate Courtney, a mountain biker, that were recorded about 2 months apart in 2020.

Episodes were divided into 5-minute documents, and topic frequency plotted throughout the course of the interview. Although noisy, there are a few common themes that appear. One is that episodes are bookended with informal chat at the beginning and end of the episode, and there is occasional informal chat in the middle of the episode. In both interviews. The first substantial portion is focused on the guest’s specific discipline. In Justin’s case this was road cycling, and in Kate’s case, MTB. Interestingly, immediately after that first discipline-specific discussion, the conversation shifted to strength. It makes sense that this is part of the conversation since it’s an important part of athlete training, but interesting nonetheless that it occurs at the same relative location in both interviews. Also in both interviews there are multiple blocks of time focused on goals and in both cases there is one occurrence in the early/ mid portion of the interview, then a return to the same topic towards the end of the interview. In light of how long-form athlete interviews are conducted, this intuitively makes sense that early discussion is about sport specifics and tends later to talk about the future, goals, and ambitions. While these are only a few cases and may be coincidental (I didn’t run statistics on these data), it does help frame the general trend of an interview and identify how topics change throughout the course of the episode.

<img src="images/NLP_podcast/Interviews.png?raw=true"/>

### Conclusions

Topic modeling from podcast transcripts represents a new way of probing information in audio form. Given the abundance of audio available, using NLP to identify when topics occur in an audio recording could be a valuable tool for searching within audio. While this model was optimized for a very specific use case, a narrow domain with a relatively small number of different topics, it demonstrates the power of NLP to extract information from a body of text. My next steps in this project would be developing a search tool to recommend segments with a user-specified topic distribution. For example, searching for documents that contain topics MTB racing and nutrition, or road racing and equipment. Additionally, adding a key word parameter to the topic search could help further narrow results for highly specific search cases. 

