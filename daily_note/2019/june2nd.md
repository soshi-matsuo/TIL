# Programming
## Python  
STTのAPI操作例  
```python
from ibm_watson import SpeechToTextV1
from os.path import join, dirname
# init STT endpoint
speech_to_text = SpeechToTextV1(   
 iam_apikey='{apikey}',
 url='{url}'
)
# add corpus to language model
with open(join(dirname(__file__), './.', 'corpus1.txt'), 'rb') as corpus_file:
speech_to_text.add_corpus(
'{customization_id}',
'corpus1',
corpus_file
)
# train language model
speech_to_text.train_language_model('{customization_id}')
```
