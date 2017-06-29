# Sentiment Analysis Using Google Cloud SDK

In this blog, I will give the tutorial for sentiment analysis for text files using google cloud API.

Similar as Google Speech API. We also have two methods for NLP (sentiment analysis) using google Language API.
***
## Using python for NLP via Google Language Cloud

### 1. Git Clone the google cloud python api 
Git clone from link https://github.com/JiaxiHe/python-docs-samples. 

```
$ git clone https://github.com/GoogleCloudPlatform
/python-docs-samples.git
$ cd python-docs-samples/language/sentiment
```

### 2. Sentiment Analysis
The python file for sentiment score analysis is under path **python-docs-samples/language/sentiment**. We can use the Mac Bash.
```
$ cd python-docs-samples/language/sentiment
$ python sentiment_analysis.py path/textfile.txt
```
can get the following,
```
Sentence 0 has a sentiment score of 0.4
Sentence 1 has a sentiment score of -0.3
Sentence 2 has a sentiment score of -0.3
Sentence 3 has a sentiment score of 0
Overall Sentiment: score of 0 with magnitude of 1.1
```

Or we can diggding into this python file in **python-docs-samples/language/cloud-client/v1/snippets.py** and writing our own code to organize the above results into a dataframe. See following.
```
import argparse
from google.cloud import language
```
```
# Instantiates a client with they v1beta2 version
language_client = language.Client(api_version='v1beta2')
document = language_client.document_from_text(nlp_output, 
language='EN')

annotations = document.annotate_text(
                            include_sentiment=True,
                            include_syntax=False,
                            include_entities=False)
```
```
score = annotations.sentiment.score
magnitude = annotations.sentiment.magnitude

sentence_no = []
sentiment = []
for index, sentence in enumerate(annotations.sentences):
    sentence_sentiment = sentence.sentiment.score
    sentence_no.append(index) 
    sentiment.append(sentence_sentiment)
    print('Sentence {} has a sentiment score of {}'.format(
            index, sentence_sentiment))
    

df = pd.DataFrame({'sentenceNum': sentence_no, 'sentiment':sentiment}, 
                 columns = ['sentenceNum', 'sentiment'], index = None)
```
We can get the dataframe as following:
sentenceNum | sentiment
---|---
0 | 0.4
1 | -0.3
2 | -0.3
3 | 0


### 3. Entity Sentiment Analysis
Entity sentiment analysis means that we can get the entity (Keyword Extraction) and sentiment score for each keyword in its context. Still, we study the python file of **python-docs-samples/language/cloud-client/v1/snippets.py** and write our own code for the entity sentiment analysis.

```
# Imports the Google Cloud client library
from google.cloud.gapic.language.v1beta2 import enums
from google.cloud.gapic.language.v1beta2 import
language_service_client
from google.cloud.proto.language.v1beta2 import
language_service_pb2
from google.cloud import language
import sys
import pandas as pd
from pandas import Series, DataFrame
import six  #provides utility functions for smoothing
#over the differences between the Python versions

```

```
language_client = language_service_client.
LanguageServiceClient()
document = language_service_pb2.Document()
document.content = nlp_output.encode('utf-8')
document.type = enums.Document.Type.PLAIN_TEXT
encoding = enums.EncodingType.UTF32
result = language_client.analyze_entity_sentiment
(document, encoding)
```

Then we got the result in json format.

***
## Use the Mac Bach to request for Google Cloud Language API

### 1. Prepare Json request file 'request.json'
```
{
"encodingType": "UTF8",
"document": {
  "type": "PLAIN_TEXT",
  "content": "Tired of everything and I wanna escape!!"
    }
}
```
In mac Bash, we first setup the authenticated environment.
```
gcloud auth application-default print-access-token
```
Then we can do sentiment analysis request.
```
curl -s -k -H "Content-Type: application/json" 
-H "Authorization: Bearer access-token"
https://language.googleapis.com/v1beta2/documents:analyzeSentiment
-d@desktop/google-nlp/request.json |pbcopy
```
similar, the entity sentiment analysis request.
```
curl -s -k -H "Content-Type: application/json" 
-H "Authorization: Bearer access-token"
https://language.googleapis.com/v1beta2/documents:analyzeEntitySentiment
-d@desktop/google-nlp/request.json |pbcopy
```
We have the json format outcome.
All Done!