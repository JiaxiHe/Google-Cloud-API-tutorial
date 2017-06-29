 # Use google Cloud Speech API for Speech Recognition
 
 There are two ways for Speech Recognition (via Google Cloud Speech):
 * **First Method:** use the python API with Google Speech Machine Learning, the github address for the Google Cloud is under website https://github.com/JiaxiHe/python-docs-samples, the speech recognition package is in the Speech file https://github.com/JiaxiHe/python-docs-samples/tree/master/speech/cloud-client.
 * **Second Method:** Use the Mac Bash to request for the Google Cloud Machine Learning using json-request-file. In this way, it would be faster.
 
For both methods, we need to install the Google Cloud SDK from the link https://cloud.google.com/sdk/. Also, both methods requires Autheication setup.

Authentication is typically done through Application Default Credentials, which means you do not have to change the code to authenticate as long as your environment has credentials. You have a few options for setting up authentication:
1. When running locally, use the Google Cloud SDK
**gcloud auth application-default login**.
This is what we need to do for the **Second Method**.

2. When running on App Engine or Compute Engine, credentials are already set-up. We have a python env (**Google Shell**) on the google cloud platform.

3. You can create a **Service Account key file**. This **service account key file** can be created under the API manager from the google cloud platform console. This **service account key file** can be used to authenticate to Google Cloud Platform services from any environment. To use the file, we need to set the GOOGLE_APPLICATION_CREDENTIALS environment variable to the path to the key file. Write the following common line in the Mac Bash 
**export GOOGLE_APPLICATION_CREDENTIALS = path/service_account.json**


After the authentication setup. Let's start with the First method.
***

## Use Python API for Speech Recognition
### 1. pip installation for Dependencies

```
brew install portaudio

pip install pyAudio

pip install SpeechRecognition
```

### 2. setup the authentication environment using service account key file. 
Detailed explanation of authenticated environment setup will be given later in the next method.

### 3. Usages
Git Clone the google cloud python api from its github link  https://github.com/JiaxiHe/python-docs-samples. 

#### QuickStart Example

```
$ cd path/python-docs-samples/speech/cloud-client
$ python quickstart.py
```

#### Transcribe

```
$ python transcribe.py

usage: transcribe.py [-h] path

Google Cloud Speech API sample application using the REST API 
for batch processing.

Example usage:
    python transcribe.py resources/audio.raw
    python transcribe.py
    gs://cloud-samples-tests/speech/brooklyn.flac

positional arguments:
  path        File or GCS path for audio file to be recognized

optional arguments:
  -h, --help  show this help message and exit
```
This *transcribe.py* is for short audio files.

#### Transcribe async

```
$ python transcribe_async.py

usage: transcribe_async.py [-h] path

Google Cloud Speech API sample application using the REST API 
for async batch processing.

Example usage:
    python transcribe_async.py resources/audio.raw
    python transcribe_async.py
    gs://cloud-samples-tests/speech/vr.flac

positional arguments:
  path        File or GCS path for audio file to be recognized

optional arguments:
  -h, --help  show this help message and exit
```
This *transcribe_async.py* is what I used for 30mins audio file transcribe processing. It differs the *transcribe.py* with extra *retry_count* and *operation.poll* for long audio files.

#### Transcribe Streaming

```
$ python transcribe_streaming.py

usage: transcribe_streaming.py [-h] stream

Google Cloud Speech API sample application using the streaming 
API.

Example usage:
    python transcribe_streaming.py resources/audio.raw

positional arguments:
  stream      File to stream to the API

optional arguments:
  -h, --help  show this help message and exit
```

The Code for an audio file on my google cloud platform storage mybucket2017 (++gs://mybucket201706/++ set as public access) is listed as following. 

```
import io
import time
```

```
from google.cloud import speech
speech_client = speech.Client()

audio_sample = speech_client.sample(
    source_uri='gs://mybucket201706/1.flac',
    encoding='FLAC',
    sample_rate_hertz=16000)

operation = audio_sample.long_running_recognize('en-US')

retry_count = 2400
while retry_count > 0 and not operation.complete:
    retry_count -= 1
    time.sleep(2)
    operation.poll()

if not operation.complete:
    print('Operation not complete and retry limit reached.')

alternatives = operation.results
```

```
nlp_output = ' '
for alternative in alternatives:
    print('talking: {}'.format(alternative.transcript))
    print('\n')
    print('Confidence: {}'.format(alternative.confidence))
    print('\n')
    
    nlp_output += '  {}'.format(alternative.transcript)
    nlp_output += '.\n'
print(nlp_output)
```
***
## Use the Mac Bash and Json Request File for Speech Recognition 

### 1. Install the Google cloud SDK
using the link https://cloud.google.com/sdk/

### 2. Login and create a new project 

Login the google cloud platform via mac bash

```
$ gcloud auth application-default login
```

Create a project on the console page by entering in *Project Name* and *Project ID*

```
Project name: SpeechRecognition
Project ID: speechrecognition2017

```

### 3. Initalization
```
gcloud init
gcloud init --skip-diagnostics 
```
Choose the configuration that satifing your needs.
I have the following configuration settings from last time login:
```
Settings from your current configuration [default] are:
compute:
  region: asia-southeast1
  zone: asia-southeast1-a
core:
  account: immersive@speechrecognition2017.iam.gserviceaccount.com
  disable_usage_reporting: 'False'
  project: speechrecognition2017

Pick configuration to use:
 [1] Re-initialize this configuration [default] 
 with new settings 
 [2] Create a new configuration
 [3] Switch to and re-initialize existing configuration: 
 [test1]
Please enter your numeric choice:
```
Select [1] here, we get the following:
```
Choose the account you would like to use to perform operations for 
this configuration:
 [1] example-service@jiaxiproject1.iam.gserviceaccount.com
 [2] immersive@speechrecognition2017.iam.gserviceaccount.com
 [3] jiaxihe1989@gmail.com
 [4] san-208@jiaxiproject1.iam.gserviceaccount.com
 [5] Log in with a new account
Please enter your numeric choice: 
```
[1] and [4] are from another project *jiaxiproject1*, we ignore them and choose our account [3].

```
Pick cloud project to use: 
 [1] speechrecognition2017
 [2] Create a new project
Please enter numeric choice or text value (must exactly 
match list item): [1]
```
If you just create the project, here may asked for enable API for your project. After that, we need to configure the compute engine.

```
Do you want to configure Google Compute Engine 
(https://cloud.google.com/compute) settings (Y/n)? Y
```
```
Which Google Compute Engine zone would you like to use as 
project default?
If you do not specify a zone via a command line flag while 
working with Compute Engine resources, the default is assumed.
 [1] asia-east1-b
 [2] asia-east1-a
 [3] asia-east1-c
 [4] asia-northeast1-b
 [5] asia-northeast1-a
 [6] asia-northeast1-c
 [7] asia-southeast1-b
 [8] asia-southeast1-a
 [9] australia-southeast1-b
 [10] australia-southeast1-c
 [11] australia-southeast1-a
 [12] europe-west1-b
 [13] europe-west1-d
 [14] europe-west1-c
 [15] europe-west2-b
 [16] europe-west2-c
 [17] europe-west2-a
 [18] us-central1-a
 [19] us-central1-f
 [20] us-central1-b
 [21] us-central1-c
 [22] us-east1-d
 [23] us-east1-b
 [24] us-east1-c
 [25] us-east4-c
 [26] us-east4-a
 [27] us-east4-b
 [28] us-west1-a
 [29] us-west1-c
 [30] us-west1-b
 [31] Do not set default zone
Please enter numeric choice or text value (must exactly match list 
item):  [8] 

```
Since I am in Australia, therefore, I select [8]. Now we finished the configuration part. 

### 4. Enable your API 
Here we enable the Speech API from the console for our speech recognition. If you are the first time to this environment, in the **Initialization** part, it will ask for the **Y/n** of enable API, enter **Y**.

### 5. Create API Credentials
In console, under **API Manager/Credentials**, click **create credentials**. We have three options:

1. API key
2. OAuth client ID
3. Service account key

We need to create all of them. 

1. There will be no misunderstanding in creating an API key.

2. For OAuth client ID, I set the product name as "Jiaxi"(my own name) and client ID as "client".

3. For Service account key, I create a new service account with name "immersive"(my company name) and i set my role to this project as owner. we will get a json file (**service_account_key.json**) after create the Service account key. Download and save it to a **path**.

### 6. Setup Authentication Environment

Then active service account key from the mac bash. This is very important.
```
$ gcloud auth activate-service-account --key-file
= path/service_account_key.json
```
setup the authentication environment using service account key file

```
export GOOGLE_APPLICATION_CREDENTIALS
= path/service_account_key.json
```

### 7. Access Token
In the Mac Terminal:
```
$ gcloud auth application-default print-access-token
```

### 8. Prepare Json request file

We have a example json request file named **sync-request.json** with uri contains the audio file as following:
```
{
  "config": {
      "encoding":"FLAC",
      "sampleRateHertz": 16000,
      "languageCode": "en-US"
  },
  "audio": {
      "uri":"gs://cloud-samples-tests/speech/brooklyn.flac"
  }
}
```
### 9. Curl operation for transcribe
As we have the access-token and request file **sync-request.json**, we can use curl commond line to call the google cloud api and transcibe audio file.
```
curl -s -k -H "Content-Type: application/json"  
-H "Authorization: Bearer access_token" 
https://speech.googleapis.com/v1/speech:recognize  
-d@path/sync-request.json
```

*Note that, the path for the service_account_key.json and the sync_request.json may not the same* 

The above **8** and **9** is a sample example and the audio file only lasts seconds. Next, we will work on long audio recognition.

### 10. Prepare audio files
To use the google cloud speech, we have following two steps for long time audio files.
1. tansfer the audio into .flat format using the link http://audio.online-convert.com/ with the following setting (or as you desired):
```
Change bit resolution: 16 Bit
Change sampling rate: 16000HZ
Change audio channels: mono

```
2. Create a bucket on google cloud platform under Storage/browser. And upload your audio file in .flat format to this bucket. I named my bucket "mybucket2017".

If the audio file is long (i.e. 30mins duration), it asks you to upload the audio to google cloud storage, otherwise run error.

The uri to your audio file in my bucket is ++gs://mybucket201706/++

### 11. Prepare json file

The following is my **request1.json**.
```
{
  "config": {
      "encoding":"FLAC",
      "sampleRateHertz": 16000,
      "languageCode": "en-US"
  },
  "audio": {
      "uri":"gs://mybucket201706/1.flac"
  }
}
```
### 12. Curl operation for transcribe
Two steps for long duration audio transcribe.
1. **Curl operation for long speech recognition**
```
curl -s -k -H "Content-Type: application/json" 
-H "Authorization: Bearer access_token" 
https://speech.googleapis.com/v1/speech:longrunningrecognize  
-d@path/request1.json
```
will get a name code as following:
```
{
  "name": "8443841777443483934"
}
```

2. **Extract the Json output from the name obtained in step 1**
```
curl -s -k -H "Content-Type: application/json" 
-H "Authorization: Bearer access_token" 
https://speech.googleapis.com/v1/operations/name
```

where *name* is the numerical code that we get in **step 1**. 

Since it may take some time for trancsribing a long audio file. When you **curl** for the *https://speech.googleapis.com/v1/operations/name* in the Mac bash, it may undergoing the transcribing and you will have the following output:

```
{
  "name": "8443841777443483934",
  "metadata": {
    "@type": "type.googleapis.com/google.cloud.speech.v1.LongRunningRecognizeMetadata",
    "progressPercent": 50,
    "startTime": "2017-06-28T04:39:13.587970Z",
    "lastUpdateTime": "2017-06-28T04:42:31.723689Z"
  }
}
```
From the above, we will know how much percentage of the audio file has been processed. You can repeate the above **Curl** operation until it is all done. Then we can use the following line to copy the output transcribing file in json format.

```
curl -s -k -H "Content-Type: application/json" 
-H "Authorization: Bearer access_token" 
https://speech.googleapis.com/v1/operations/name |pbcopy
```
### 13. Extract Json file 
Till now, we got the Json output file having the following format:
```
{
  "name": "8443841777443483934",
  "metadata": {
    "@type": "type.googleapis.com/google.cloud.speech.v1.
    LongRunningRecognizeMetadata",
    "progressPercent": 100,
    "startTime": "2017-06-28T04:39:13.587970Z",
    "lastUpdateTime": "2017-06-28T04:45:54.923460Z"
  },
  "done": true,
  "response": {
    "@type": "type.googleapis.com/google.cloud.speech.v1.
    LongRunningRecognizeResponse",
    "results": [
      {
        "alternatives": [
          {
            "transcript": "my Mansion energy licks baking",
            "confidence": 0.7339176
          }
        ]
      }
      ...
      {
        "alternatives": [
          {
            "transcript": " that's a problem thank you thanks 
            bye bye",
            "confidence": 0.5728518
          }
        ]
      }
    ]
  }
}
```
Save the json content as a json file, I named it **1.json** under path '*/output-json/*'. Following is my python code to extract the content of the json file into a .txt file.
```
import json

text_file = open("output-text/1.txt", "w")

with open('output-json/1.json') as json_file:  
    data = json.load(json_file)
    result = data['response']['results']
    
    for p in result:
        text_file.write('talking: ')
        text_file.write(p['alternatives'][0]['transcript']) 
        text_file.write('\n\n')
        
        #print('Talking:')
        #print(p['alternatives'][0]['transcript'])
        #print(p['alternatives'][0]['transcript'])
        #print('\n')

text_file.close() 
```

All done!