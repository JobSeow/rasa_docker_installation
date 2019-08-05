# rasa_docker_installation
A brief document on how to get a rasa container running on docker
**Setup Rasa for Docker:**
1. Find the [rasa](https://hub.docker.com/r/rasa/rasa) repository on docker hub and pull the image in your desired directory. 

```docker pull rasa/rasa ```

Next, go to the desired directory where you wish to store your rasa files and execute the next command. 

```docker run -v "$(pwd)":/app rasa/rasa init --no-prompt```
 
This command mounts your current working directory to the working directory in the Docker container. This means that files you create on your computer will be visible inside the container, and files created in the container will get synced back to your computer.

After pulling the image and initializing it, we can start a rasa/rasa container. Simple execute the following command. 

``` docker run -dp 5005:5005  -v "$(pwd)"/models:/app/models   rasa/rasa run --enable-api```


2. After setting up the rasa container, we can start to training through either RasaUI (From step 2 on rasaUI docker hub page) or through the command line. The above instructions are sufficient to utilize RasaUI. The following content is to explain the important training files and their purposes.

3. nlu.md
Training for the nlu component can be done by editing the nlu markdown file. Under each intent header, you can specify the expressions used to specify that intent. For example, under the greet intent, expressions used could include: "hey", "hello"
```
## intent:greet
- hey
- hello
- hi
- good morning
- good evening
- hey there
```
   You can also specify the entities values and the entity types that they fall under in your training file. As seen below, the words specified in the squared brackets signify the `[entity value]`, whereas the words in the bracket specifies the entity type as well as its corresponding synonym `(entity type : synonym)`. It is not necessary that all entity values  fall under a particular synonym. In this case the brackets only specify the entity type. `(entity type)`
```
## intent:approve
- approve [tender](procurement approach:Tender)
- approve [svp](procurement approach:Small Value Purchase)
- how about approving [quotation](procurement approach) as well

## intent:introduce
- hey, i am [Tom](PERSON)
- hello, i am [Thomas](PERSON)
```

After specifying the nlu training file, we can move on to the training files used for rasa core. They are mainly, the stories.md and domain.yml file.

4. Stories.md
Your stories file simulate how a normal conversation is like. Rasa looks at this and learns from the conversations. Your stories file can look like the example below. 
```
## Generated Story 8013105630479704408
* greet
- utter_greet
* approve_svp{'procurement approach': 'Small Value Purchase'}
- action_approve

## Generated Story -7732169539704474238
* greet
- utter_greet
* approve_tender{'procurement approach': 'Tender'}
- action_approve

## Generated Story -6643313648022301209
* greet
- utter_greet
* approve_quotation{'procurement approach': 'Quotation'}
- action_approve
```

5. Domain.yml
Your domain file is the file that specifies the universe in which your assistant operates. It specifies the intents, entities, slots, and actions your bot should know about.

```
actions:
- action_approve
- action_define
- utter_approve_quotation
- utter_approve_svp
- utter_approve_tender
- utter_cheer_up
entities:
- definitions
- procurement approach
intents:
- approve
- greet
- define
- ask_weather
templates:
utter_approve_quotation:
- text: To approve quotation
utter_approve_svp:
- text: To approve svp
utter_approve_tender:
- text: to approve tender
utter_cheer_up:
- text: 'Here is something to cheer you up:'
utter_greet:
- text: 'Hi, how are you?'
```
6. Config.yml
Your config file is used to specify the configurations used for training. 
Pipelines- Choose from two (pretrained_embeddings_spacy, supervised_embeddings)
Policies- KerasPolicy, EmbeddingPolicy, MemoizationPolicy, MappingPolicy
Epochs- Epochs are how many times rasa passes the entire dataset forward and backward through the neural network.
```
language: en
pipeline: supervised_embeddings

policies:
- name: KerasPolicy
  epochs: 2000
- name: MemoizationPolicy
- name: MappingPolicy

```
Some of the more important policies:
KerasPolicy/ EmbeddingPolicy:
Main policies that predict the next action to be returned. 

Memoization policy:
The MemoizationPolicy just memorizes the conversations in your training data. It predicts the next action with confidence 1.0 if this exact conversation exists in the training data, otherwise it predicts None with confidence 0.0.

Fallback policy
The FallbackPolicy invokes a fallback action if the intent recognition has a confidence below nlu_threshold or if none of the dialogue policies predict an action with confidence higher than core_threshold.
