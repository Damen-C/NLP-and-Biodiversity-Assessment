# NLP-and-Biodiversity-Assessment

In this project, I intend to leverage advanced **NLP techniques** to analyze the insights provided by experts in response to regional assessments carried out by **The Intergovernmental Science-Policy Platform on Biodiversity and Ecosystem Service (IPBES)**. The goal of this project is to explore the contributions that NLP can offer to the realm of biodiversity and ecosystem evaluation.

## Visualization 
**Figure 1:** A Semantic Network of Keyphrases Extracted  
![](https://github.com/Damen-C/NLP-and-Biodiversity-Assessment/blob/main/Network.png?raw=true)

**Figure 2:** Phrases Associated with Positive Sentiments  
![](https://github.com/Damen-C/NLP-and-Biodiversity-Assessment/blob/main/Positive_words.png?raw=true)

**Figure 3:** Phrases Associated with Negative Sentiments  
![](https://github.com/Damen-C/NLP-and-Biodiversity-Assessment/blob/main/Negative_words.png?raw=true)

## Summary of Project 
The Intergovernmental Science-Policy Platform on Biodiversity and Ecosystem Service (IPBES), an intergovernmental body founded in 2012, has taken significant role in examining biodiversity trends and nature's contributions at regional and global scales. Notably, in 2015, IPBES initiated four regional assessments including Africa, the Americas, Asia Pacific, and Europe and Central Asia. These endeavors aim to strengthen the link between science and policymaking at the regional level.

A cornerstone of these assessments' credibility lies in the peer review overseen by the Multidisciplinary Expert Panel (MEP) of IPBES. Expert commentary offers insights into regional concerns and potential areas for practical action. However, a gap exists: these expert comments have not been systematically studied, either qualitatively or quantitatively.

This paper proposes an innovative approach: leveraging Natural Language Processing (NLP)—a specialized branch of Machine Learning targeting language understanding—to undertake sentiment analysis and contextual semantic network examination of the expert comments, focusing on the Asia Pacific region. Using NLP techniques promises to shed light on predominant expert sentiments and key thematic concerns, potentially guiding future research and policy direction in biodiversity and ecosystem services.

**Five key words for this project:**
1. SDGs
2. Biodiversity assessment
3. Natural Language Processing
4. Sentiment Analysis
5. Network Analysis

## Data Collection
I first go to the following [website](https://www.ipbes.net/assessment-reports/asia-pacific) to download PDF files under the Comment second review section. I then use the pdfplumber package in Python to convert PDF files to XLSX files for further analysis.

## Sentiment Analysis with BERT
For tokenizing the comments, I used an auto tokenizer from the pre-trained model **“nlptown/bert-base-multilingual-uncased-sentiment”**.  The bert-base-multilingual-uncased model is finetuned for sentiment analysis on product reviews in six languages: English, Dutch, German, French, Spanish, and Italian. It predicts the sentiment of the experts comment between 1 and 5, 1 represents the most negative and 5 represents the most positive. See [here](https://huggingface.co/nlptown/bert-base-multilingual-uncased-sentiment).

## Visualize the Frequency of Keyphrases and their Importance on Sentiments using WordCloud: 
I first preprocess the comments by removing URLs and punctuation. This ensures that the text data is cleaner and more suitable for further analysis.

I use **keyphrase-vectorizers package** in Python to extract key phrases from every comment. Instead of using n-gram tokens of a pre-defined range, this method extracts keyphrases from text documents using part-of-speech tags to compute document-keyphrase matrices.See [here](https://pypi.org/project/keyphrase-vectorizers/). 

To show the importance of each keyphrase on the sentiment score, I use a linear regression for each keyphrase against the sentiment scores. For each keyphrase, I run a simple linear regression model where:  
The independent variable is the presence (or absence) of the keyphrase (binary: 1 or 0).  
The dependent variable is the sentiment score.  
The Ordinary Least Squares (OLS) method is used to fit the linear regression model.  

For each keyphrase, after fitting the linear regression model, I extract the coefficient of the keyphrase (coef), which indicates the change in the sentiment score for a one-unit change in the keyphrase (i.e., its presence vs. absence). And the p-value (p_value), which tests the hypothesis that the keyphrase has no effect on the sentiment score. A small p-value (typically ≤ 0.05) indicates that the keyphrase is statistically significant in predicting the sentiment score. Keyphrases wirth p-values larger than 0.05 are removed. I extract the 100 keyphrases with the largest magnitude of negative and positive coefficients respectfully.  

**To interpret the coefficient:** 
A negative coefficient suggests that the presence of the keyphrase is associated with a decrease in sentiment score, implying a negative sentiment.  
A positive coefficient suggests that the presence of the keyphrase is associated with an increase in sentiment score, implying a positive sentiment.  
The magnitude (absolute value) of the coefficient indicates the strength or intensity of this association. The larger the absolute value, the stronger the relationship between the word and the sentiment score. 

I then use the extracted 200 keyphrases as keys to count the frequency each keyphrase appears in all comments.   

To visualize the frequency of keyphrases and their importance on sentiment scores, I use the Python package **wordcloud**. In this word cloud, two aspects are considered: **the size and the color**. The size of keyphrase represents its coefficient – the larger the keyphrase is, the higher the influence of that phrase on the sentiment. The color of keyphrase represents its frequency, with a darker color indicating a higher frequency.

## Network Analysis:
I use keyphrase-vectorizers library in Python to extract key phrases from the comments. 
The extracted candidate key phrases are subsequently passed to KeyBERT for embedding generation and similarity calculation. See [here](https://github.com/TimSchopf/KeyphraseVectorizers). 
What this KeyBERT does is that it creates keywords and keyphrases that are most similar to a corpus. 

To construct a semantic network from the key phrases extracted from the IPBES expert comments, I leveraged BERT embeddings to capture semantic relationships between these key phrases. 

I first utilized KeyBERT to extract the top 50 salient key phrases from the aggregated comments text. This provided an initial set of important topics. 

Then, to obtain more meaningful embeddings, I extracted a contextual window of text around each key phrase occurrence and computed embeddings on this contextual snippet. Specifically, for each key phrase match, I identified the start index of the match in the overall text. I then defined a contextual window spanning n words before and after the key phrase. For example, with a window size n=20, and a key phrase starting at index 100 in the text, I would extract indices 80 to 120 as the contextual window. This extracts some words before and after the key phrase to incorporate the local context.

The appropriate window size is an empirically tuned hyperparameter. A smaller window may not capture enough context, while a larger window starts to include less relevant text. I experimented with window sizes between 10-30 and found 20 to be a reasonable size for our text data based on manual examination. The contextual window text snippet is then fed into the pretrained BERT model to obtain a high dimensional embedding vector. This embedding encodes the semantic meaning of the keyphrase in its local context. The contextual embeddings allow each keyphrase occurrence to have a unique vector based on how it is used, rather than a single embedding for each keyphrase type. This provides greater resolution in quantifying semantic similarities between keyphrase usages.

I then computed a cosine similarity matrix between all contextual keyphrase embeddings to quantify semantic similarity. Highly similar keyphrase pairs were connected in a weighted networkx graph, with an empirically chosen threshold of 0.7. Node weights were assigned based on keyphrase importance scores from KeyBERT. Edge weights were defined as the cosine similarity between connected nodes. Finally, the graph was exported into two CSV files representing nodes and edges. This facilitates analysis and visualization using Gephi.
