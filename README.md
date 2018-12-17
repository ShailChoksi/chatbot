# lichess-chatbot
Chatbot to connect to Lichess bots

# Setup
```
virtualenv .venv -p python3
pip install -r requirements.txt
python -m spacy download en
```

# How to create a bot
- This makes use of rasa_nlu and rasa_core
- rasa_nlu uses an intent classification neural network with a crf/spacy/duckling entity extractor
- rasa_core uses a recurrent neural network that creates the best path(action) to follow in a conversation given the intents and entities

## Training the NLU (rasa_nlu)
- Create the training data in src/data
- It should contain the intent(s) name and several data points (ideally 20 or more) per intent. E.g.:
```
## intent:smalltalk_greet
- Hey!
- Hello
- Whats up?
- Hi
```
- For more info: https://rasa.com/docs/nlu/
- Once the data is created, you will need to create a config, which has a pipeline. For most cases this will suffice:
```
pipeline:
- name: "nlp_spacy"
  model: "en"
- name: "tokenizer_spacy"
- name: "intent_featurizer_spacy"
- name: "intent_featurizer_ngrams"
- name: "ner_spacy"
- name: "intent_featurizer_count_vectors"
- name: "intent_classifier_tensorflow_embedding"
  epochs: 30
  "num_hidden_layers_a": 2
  "hidden_layer_size_a": [32, 16]
  "num_hidden_layers_b": 0
  "hidden_layer_size_b": []
  "batch_size": [16, 32]
  # embedding parameters
  "embed_dim": 30
  # regularization parameters
  "C2": 0.02
  "droprate": 0.2
```
- Save it under src/configs
- To train the model run: ```python -m rasa_nlu.train --data src/data --config configs/your_config.yml --path models/ --fixed_model_name NAME_OF_MODEL```

## Training rasa_core
- Create a stories.md file under src/stories/model_name
- You can follow the quick start tutorial here: https://rasa.com/docs/core/quickstart/#write-stories
- Create a domain.yml file under src/stories/model_name
- You can follow the quick start tutorial here: https://rasa.com/docs/core/quickstart/#define-a-domain
- Train rasa_core: ```python -m rasa_core.train -d src/stories/model_name/domain.yml -s src/stories/model_name/stories.md -o models/dialogue```

## Running the full bot
- ```python -m rasa_core.run -d models/dialogue -u models/ --enable_api --auth_token thisismysecret -o out.log```
- You can now send POST HTTP requests
```
curl -XPOST http://localhost:5005/webhooks/rest/webhook -d '{"message": "<your text to parse>"}'
``` 