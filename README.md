Chat XBlock
===========

An XBlock that allows learners to chat with a bot, where the bot follows
a script and the learner can choose among possible responses.

Authors define the script as a set of "steps".
See [YAML Configuration](#yaml-configuration) section for more info.


Installation
------------

Install the requirements into the Python virtual environment of your
`edx-platform` installation by running the following command from the
root folder:

```bash
$ pip install -r requirements.txt
```


Enabling in Studio
------------------

You can enable the Chat XBlock in Studio through the Advanced
Settings.

1. From the main page of a specific course, navigate to `Settings ->
   Advanced Settings` from the top menu.
2. Check for the `Advanced Module List` policy key, and add
   `"chat"` to the policy value list.
3. Click the "Save changes" button.


YAML Configuration
------------------

The steps list is defined as a YAML sequence of mappings whose key represents
the step id and the value is a nested mapping with messages and optional responses.
We usually use `step1`, `step2`, ... in our examples, but please note that step
ids are arbitrary.

Each response is a mapping of response text to the next step id.

The chat is complete when the users reaches a step that has no responses,
or when the user selects a response which links to a non-existent step id.

A valid YAML steps sequence looks like this:

```yaml
- step1:
    messages: What is the sum of 1 and 1?
    responses:
        - 2: step2
        - 3: step3
- step2:
    messages: Yep, that's correct! Good job.
- step3:
    messages: Hmm, no, it's not 3. (It's less.) Would you like to try again?
    responses:
        - Yes please: step1
        - No thanks: null
```

### Sequence of Bot Messages

Each step can contain more than one bot messages. Multiple bot messages are displayed
in succession.

```yaml
- step1:
    messages:
        - Let's do some math!
        - We will start with simple sums first.
        - What is the sum of 1 and 1?
    responses:
        - 2: step2
        - 3: step3
- ...
```

### Step without Bot Messages

Steps that contain an empty message attribute are valid and will display user
responses immediatelly without transferring control to the bot, for example:

```yaml
- step1:
    messages: Hello, would you like to learn some math?
    responses:
        - Yes please: step2
        - No thanks: null
- step2:
    messages: []
    responses:
        - Let's do addition first: step3
        - Let's do multiplication first: step4
- ...
```

### Images in Bot Messages

It is possible to add an optional image (with alternative text) to a step.

```yaml
- step1:
    messages: What is the sum of 1 and 1?
    image-url: http://example.com/sum.png
    image-alt: Graphic representation of this sum.
    responses:
        - 2: step2
        - 3: step3
- ...
```

### Bot Message Randomization

If a step contains a nested list of messages, a single message from the nested
list is picked randomly each time the step is displayed. If the same step is
displayed multiple times, it will pick a different message each time.

In the example below, the first time the first step is displayed, the bot will
pick one of "What is the sum of 1 and 1?" and "What is 1+1?" at random. If the
user gets it wrong the first time, and then answers "Yes please" to try again,
the first step is displayed again, but this time the other message will be
chosen. If the user gets it wrong again and goes back to step1 the third time,
the message will be picked randomly again.

```yaml
- step1:
    messages:
        - ["What is the sum of 1 and 1?", "What is 1+1?"]
    responses:
        - 2: step2
        - 3: step3
- step2:
    messages: Correct.
- step3:
    messages: Wrong. Would you like to try again?
    responses:
        - Yes please: step1
        - No thanks: null
```

### Multiple Bot Personas

A single bot persona interacts with the user by default, but you can define
multiple bot personas with different profile images.

In order to use multiple bot personas, you have to define profile images for
each bot persona as described in
[Multiple Bot Image Configuration](#multiple-bot-image-configuration).

You can then specify which message belongs to which bot persona in YAML
configuration:

```yaml
- step1:
    messages:
        - alice: Hello, my name is Alice!
        - bob: And I am Bob.
        - alice: We will help you learn about quantum entanglement.
    responses:
        - Great: step2
- ...
```


Bot Profile Image URL Configuration
-----------------------------------

A default bot profile image is provided by this xblock, but you can use a
different image by entering its URL in the "Bot profile image URL" field
in the Studio. You can use an absolute URL or a relative reference to images
uploaded in the Studio (for example `/static/myimage.png`).

### Multiple Bot Image Configuration

When using [multiple bot personas](#multiple-bot-personas), you have to define
the images for each bot persona as a YAML mapping of bot ids to image URLs,
for example:

```yaml
alice: /static/alice-profile-img.jpg
bob: /static/bob-profile-image.jpg
```


Testing
-------

Inside a fresh virtualenv, `cd` into the root folder of this repository
(`xblock-chat`) and run

```bash
$ pip install -r requirements-test.txt
$ pip install -r $VIRTUAL_ENV/src/xblock-sdk/requirements/base.txt
$ pip install -r $VIRTUAL_ENV/src/xblock-sdk/requirements/test.txt
$ pip install -r $VIRTUAL_ENV/src/xblock/requirements.txt
```

You can then run the entire test suite via

```bash
$ python run_tests.py
```


Necessary changes
-----------------

1. This XBlock displays the profile image of the user in the chat interface. For this feature to work on a devstack, it's necessary to add:

```python
MEDIA_ROOT = '/edx/var/edxapp/media'
```

to `edx-platform/lms/envs/private.py`

2. In order to make the chat interface responsive it's necessary to apply [this fix](https://github.com/open-craft/edx-platform/commit/2a1cf699452ae567bcb3caeb507760f29f1df830) to the LMS chromeless courseware template.
