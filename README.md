## snips-app-joke-tuto
[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/CoorFun/snips-app-joke-tuto/blob/master/LICENSE)

This is an example snips APP based on the [official python template]

## Tutorial

In this example, we will make a joke app by using this rich template. It shall either fetch a random joke or use one from a given category. The designed workflow is shown below:

![Joke action interaction work flow](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-L5OxUOD7uLDGd059vYc%2F-LOSEi4mOgVZFOEdgPoe%2F-LOSIpXTLJI3AifFyYO_%2Fimage%20(1).png?alt=media&token=08e18f53-f6c6-4d09-91d4-91e584ea33ff)

## Bundle Design
To be able to get intents from natural language, the first step is to create a bundle, in other words, your assistant. For this example, we will only have one intent, which is askJoke with one slot - category.

| Intent | Slots | Description | 
| --- | --- | --- |
| askJoke | category | Fetch a joke |

> We highly recommend to those who would open source their App code repository to create a table describing the bundle design even if there's only one intent involved.

Here are all the joke categories: `explicit`, `dev`, `developer`, `development`, `movie`, `cinema`, `food`, `celebrity`, `science`, `sport`, `sporting`, `animal`, `history`, `music`, `travel`, `travelling`, `career`, `money`, `fashion`.

## Action Code

The main function of the action code is getting the category value then fetch the joke from a free API. 

> Thanks to https://api.chucknorris.io/ for providing the free joke API. 

Following are three APIs we need to use:

- ***GET: https://api.chucknorris.io/jokes/categories (List of available categories)***
- ***GET: https://api.chucknorris.io/jokes/random?category={category} (Get a joke from given category)***
- ***GET: https://api.chucknorris.io/jokes/random (Get a random joke)***

### Step 1/ Initialising an App repository

Create a Github repository, clone it to your local device. Init it with the contents [here](https://github.com/snipsco/snips-app-template-py), then you should have the files organised like mentioned in the Template organisation section. 

In this example, we named this repo as `snips-app-joke-tuto`. Then rename the `action-app_template.py` file to `action-app_joke_tuto.py`.

### Step 2/ Filling action-app_joke_tuto.py with functional code

> Since there are two pre-set sub callback functions, we will only keep one.

API request will be done by `requests` module, so this should be imported at the beginning of the code. 

```python
import requests
```

Then rename the class from `Template` to `JokeTuto`

```python
class JokeTuto(object):
```

This project will only have one intent, so only one sub callback function should be kept. Let's rename it from `intent_1_callback` to `askJoke_callback`. Also, delete 2nd sub callback `intent_2_callback`.

Once again, do not forget to do the same thing in the master callback function `master_intent_callback`. Change the intent name to `<snips_console_user_name>:askJoke`. (We have changed it to `coorfang:askJoke`)

We are now good to write functional code in the `askJoke_callback` function:

```python
# --> Sub callback function, one per intent
def askJoke_callback(self, hermes, intent_message):
    # terminate the session first if not continue
    hermes.publish_end_session(intent_message.session_id, "")
    
    # action code goes here...
    good_category = requests.get("https://api.chucknorris.io/jokes/categories").json();

    category = None
    if intent_message.slots.category:
        category = intent_message.slots.category.first().value
        # check if the category is valide
        if category.encode("utf-8") not in good_category:
            category = None

    if category is None:
        joke_msg = str(requests.get("https://api.chucknorris.io/jokes/random")\
                                                                .json().get("value"))
    else:
        joke_msg = str(requests.get("https://api.chucknorris.io/jokes/random?category={}".format(category))\
                                                                .json().get("value"))

    # if need to speak the execution result by tts
    hermes.publish_start_session_notification(intent_message.site_id, joke_msg, "Joke_Tuto_APP")
```

The sub callback function first close the session by giving an empty string. Then it fetch the `category` list of the jokes. This will be later used to check if a detected category value is validated. Slot value is checked just afterward. 

By checking if the category is "None", we can choose to use different API requests. 

Finally, we can play the joke by using the `publish_start_session_notification` function. This is the TTS function you are supposed to use whenever you need text-to-speech.

Step 3/ Added dependency to `requirements.txt`

In this example, we have used the `requests` module. But this package is not initially installed with python2. Do not forget to add it into `requirements.txt`.

```bash
# Bindings for the hermes protocol
hermes-python>=0.1

requests
# More dependency goes here..
```

Step 4/ Get using `config.ini`

You may note that all the jokes are related to [Chuck Norris](https://en.wikipedia.org/wiki/Chuck_Norris). If you want to use someone's else name, we can add a config option to change the "protagonist" name. 

First, let's modify the `config.ini` file to have this config entity, for this example we are going to change it to Jackie Chan : )

```bash
# no section for preset values

[secret]
#empty value for secret values
protagonist=Jackie Chan
```

Then let's add a line before the joke_msg is played. 

```python
new_people =  self.config.get("secret").get("protagonist")
if new_people is not None and new_people is not "":
    joke_msg = joke_msg.replace('Chuck Norris',new_people)
```

### Step 5/ Debug and test

Use `sam` to install the test assistant, which contain the joke bundle (For the moment there is only a bundle, without any action code. 

Code will be tested locally before being bounded to the bundle.):
```bash
sam install assistant
```

Then copy the App folder to `/var/lib/snips/skill/` to test: (On Raspberry)

```bash
sudo cp -r snips-app-joke-tuto/ /var/lib/snips/skill/
```

Check if the `action-app_joke_tuto.py` and `setup.sh` has execution rights, if not:

```bash
chmod +x action-app_joke_tuto.py
chmod +x setup.sh
```

Install the virtual environment and dependencies and execute `setup.sh`:

```bash
sudo ./setup.sh 
```

Manually stop the `snips-skill-server`:

```bash
sudo systemctl stop snips-skill-server
```

Then manually start it with `-vvv` to have more detailed logs:

```bash
snips-skill-server -vvv
```

Now you are able to see the output of the skill server. 

By saying ***"Hey snips, please tell me a joke"***, you can see if there are some bugs in your code. Find and fix the existing problems. 

When there aren't any bugs displayed and everything works fine, let's move to the deploying step.

> Once everything is done, do not forget to restart the stopped `snips-skill-server`:
>
> ```sudo systemctl restart snips-skill-server```

### Step 6/ Deploy the repository with bundle

This step is easy, all the things you need to do is copying and pasting your repository address to the bundle action page. (Be sure that you select Github as the Action Type). This is shown in the screenshot below.

![Bundle](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-L5OxUOD7uLDGd059vYc%2F-LOSEi4mOgVZFOEdgPoe%2F-LOSNfABtojbnRlyqUQC%2Fimage%20(2).png?alt=media&token=5e864511-cdac-464f-ad5c-823af40daa12)

Congratulations! You have now finished you first rich code snips App!
