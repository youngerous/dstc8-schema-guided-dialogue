# Schema-Guided Dialogue State Tracking [(DSTC 8)](https://sites.google.com/dstc.community/dstc8)

**Organizers -** Abhinav Rastogi, Xiaoxue Zang, Raghav Gupta, Srinivas Sunkara,
Pranav Khaitan

**Contact -** schema-guided-dst@google.com

## Important Dates


|                                                 | Date                  |
| ----------------------------------------------- | :-------------------- |
| Task description released.                      | 06/17/2019            |
| Sample data released. Development phase begins. | 06/18/2019            |
| Single domain dataset (train + dev) released    | 07/07/2019            |
| Baseline Released                               | 07/19/2019 (expected) |
| Multi domain dataset (train + dev) released     | 07/19/2019 (expected) |
| Test phase begins.                              | 10/07/2019            |
| Entry submission deadline.                      | 10/13/2019            |
| Objective evaluation completed.                 | 10/20/2019            |


### Updates

**07/07/2019** - Train and dev sets for the
[single domain dataset](#single-domain-dataset) have been released. The baseline
system will be released with the multi domain dataset on 07/19/2019. The sample
dialogues released earlier in `train/dialogues_1.json` have been moved to
`train/dialogues_43.json`.

**06/30/2019** - Due to some unforeseen delays, the single domain dataset is
expected to be released on 07/07/2019. The other dates are unchanged.

## Introduction

Virtual assistants such as the Google Assistant, Alexa, Siri, Cortana etc. help
users accomplish tasks by providing a natural language interface to service
providers (backends/APIs). Such assistants need to support an ever-increasing
number of services and APIs. Building a dialogue system that can seamlessly
operate across all these services is the holy grail of conversational AI. The
Schema-Guided State Tracking track in the 8th Dialogue System Technology
Challenge explores dialogue state tracking in such a setting. This challenge is
aimed to encourage research on multi-domain dialogue systems that can
efficiently interface with a large number of services/APIs. We have taken a few
other practical considerations (e.g, defining a large variety of slot types, not
providing a list of all possible values for some slots etc.) while creating this
dataset to closely mimic the real world use cases.

Under the schema-guided approach, the dialogue state representation is based on
the schemas for the services under consideration (see figure below for an
example). Each dialogue in the dataset is accompanied by schemas listing a set
of user intents and slots, and a sentence describing their semantics in natural
language. The dialogue state needs to be predicted over these intents and slots.
Such a setup ensures that the system learns to recognize the semantics of
intents and slots from their descriptions, rather than just treating them as
labels, thus allowing zero-shot generalization to new schemas. Furthermore, this
approach enables the system to recognize patterns across domains (e.g, slots for
date or time are specified in a similar manner across different services)
without providing an explicit alignment between slots of different services.

![](schema_guided_overview.png)

## Data

The dataset consists of conversations between a virtual assistant and a user.
These conversations have been generated with the help of a dialogue simulator
and paid crowd-workers using an approach similar to that outlined
[here](https://arxiv.org/pdf/1801.04871.pdf). The details and code for the
dialogue simulator will be made public after the competition. **The dataset is
provided "AS IS" without any warranty, express or implied. Google disclaims all
liability for any damages, direct or indirect, resulting from the use of the
dataset.**

### Schema Representation

A schema is a normalized representation of the interface exposed by a
service/API. The schemas have been manually generated by the dataset creators.
This representation of schema has been chosen to reflect real world
services/APIs by imposing the following restrictions:

1.  **Arbitrary calls are not allowed.** A service/API only offers a limited
    number of ways to call it. E.g, a call to a restaurant search API may only
    be made once the location and cuisine are known. Although this doesn't
    impact the dialogue state tracking task directly, it has implications on the
    flow of the dialogue (e.g, the system must gather the value of all required
    slots from the user before making a service call), which participants may
    exploit.
2.  **The list of all possible entities is not accessible.** Many services don't
    expose the set of all available entities (defined as an assignment of values
    to all slots or a single row in the underlying database), and for others
    this list could be very large or dynamic. So, we don't provide a list of all
    possible entities. *This rules out approaches which represent dialogue state
    as a distribution over the set of all possible entities.*
3.  **The list of all values taken by a slot is not provided for some slots.**
    Obtaining the list of all possible values taken by some slots is not
    feasible because this could be very large (restaurant name, city etc.),
    unbounded (date, time, username etc.) or dynamic (movie, song etc.) *This
    rules out approaches which represent dialogue state as set of distribution
    over the set of all possible values of a slot, or which iterate over all
    possible slot values.* The list of all possible values is provided for some
    slots like price range, star rating, number of people etc, where it is
    natural to do so.

The schema for a service contains the following fields:

*   **service_name** - A unique name for the service.
*   **description** - A natural language description of the tasks supported by
    the service.
*   **slots** - A list of slots/attributes corresponding to the entities present
    in the service. Each slot contains the following fields:
    *   **name** - The name of the slot.
    *   **description** - A natural language description of the slot.
    *   **is_categorical** - A boolean value. If it is true, the slot has a
        fixed set of possible values.
    *   **possible_values** - List of possible values the slot can take. If the
        slot is a categorical slot, it is a complete list of all the possible
        values. If the slot is a non categorical slot, it is either an empty
        list or a small sample of all the values taken by the slot.
*   **intents** - The list of intents/tasks supported by the service. Each
    method contains the following fields:
    *   **name** - The name of the intent.
    *   **description** - A natural language description of the intent.
    *   **is_transactional** - A boolean value. If true, indicates that the
        underlying API call is transactional (e.g, a booking or a purchase), as
        opposed to a search call.
    *   **required_slots** - A list of slot names whose values must be provided
        before making a call to the service.
    *   **optional_slots** - A dictionary mapping slot names to the default
        value taken by the slot. These slots may be optionally specified by the
        user and the user may override the default value. An empty default value
        allows that slot to take any value by default, but the user may override
        it.

### Dialogue Representation

The dialogue is represented as a list of turns, where each turn contains either
a user or a system utterance. The annotations for a turn are grouped into
frames, where each frame corresponds to a single service. Each turn in the
single domain dataset contains exactly one frame. In multi-domain datasets, some
turns may have multiple frames.

Each dialogue is represented as a json object with the following fields:

*   **dialogue_id** - A unique identifier for a dialogue.
*   **services** - A list of services present in the dialogue.
*   **turns** - A list of annotated system or user utterances.

Each turn consists of the following fields:

*   **speaker** - The speaker for the turn. Possible values are "USER" or
    "SYSTEM".
*   **utterance** - A string containing the natural language utterance.
*   **frames** - A list of frames, each frame containing annotations for a
    single service.

Each frame consists of the following fields:

*   **service** - The name of the service corresponding to the frame. The slots
    and intents used in the following fields are taken from the schema of this
    service.
*   **slots** - A list of slot spans in the utterance, only provided for
    non-categorical slots. Each slot span contains the following fields:
    *   **slot** - The name of the slot.
    *   **start** - The index of the starting character in the utterance
        corresponding to the slot value.
    *   **exclusive_end** - The index of the character just after the last
        character corresponding to the slot value in the utterance. In python,
        `utterance[start:exclusive_end]` gives the slot value.
*   **actions** (system turns only) - A list of actions corresponding to the
    system. Each action has the following fields:
    *   **act** - The type of action. The list of all possible system acts is
        given below.
    *   **slot** (optional)- Aslot argument for some of the actions.
    *   **values** (optional)- A list of values assigned to the slot. If the
        values list is non-empty, then the slot must be present.
*   **state** (user turns only) - The dialogue state corresponding to the
    service. It consists of the following fields:
    *   **active_intent** - The intent corresponding to the service of the frame
        which is currently being fulfilled by the system. It takes the value
        "NONE" if none of the intents are active.
    *   **requested_slots** - A list of slots requested by the user in the
        current turn.
    *   **slot_values** - A dictionary mapping slot name to a list of strings.
        For categorical slots, this list contains a single value assigned to the
        slot. For non-categorical slots, all the values in this list are spoken
        variations of each other and are equivalent (e.g, "6 pm", "six in the
        evening", "evening at 6" etc.).

List of possible system acts:

*   **INFORM** - Inform the value for a slot to the user. The slot and values
    fields in the corresponding action are always non-empty.
*   **REQUEST** - Request the value of a slot from the user. The corresponding
    action always contains a slot, but values are optional. When values are
    present, they are used as examples for the user e.g, "Would you like to eat
    indian or chinese food or something else?"
*   **CONFIRM** - Confirm the value of a slot before making a transactional
    service call.
*   **OFFER** - Offer a certain value for a slot to the user. The corresponding
    action always contains a slot and a list of values for that slot offered to
    the user.
*   **NOTIFY_SUCCESS** - Inform the user that their request was successful. Slot
    and values are always empty in the corresponding action.
*   **NOTIFY_FAILURE** - Inform the user that their request failed. Slot and
    values are always empty in the corresponding action.
*   **INFORM_COUNT** - Inform the number of items found that satisfy the user's
    request. The corresponding action always has "count" as the slot, and a
    single element in values for the number of results obtained by the system.
*   **OFFER_INTENT** - Offer a new intent to the user. Eg, "Would you like to
    reserve a table?". The corresponding action always has "intent" as the slot,
    and a single value containing the intent being offered. The offered intent
    belongs to the service corresponding to the frame.
*   **REQ_MORE** - Asking the user if they need anything else. Slot and values
    are always empty in the corresponding action.
*   **GOODBYE** - End the dialogue. Slot and values are always empty in the
    corresponding action.

### The Dialogue State

The dialogue state is the system's estimate of the user's goal based on the
dialogue context. It is used to identify the appropriate service call to make
and to assign values to different slots required by the service. The state is
also used by the system to generate the next actions. In our setup, a separate
dialogue state is maintained for each service in the corresponding frame. All
turns in a single domain dialogue have exactly one frame. However, turns in
multi-domain dialogues may have more than one frame. More details about this
will be added after the multi domain dataset is released, however no changes
will be made to single domain dialogues.

The dialogue state is only available for user turns. Within a frame, the
dialogue state contains information about the active intent for the
corresponding service, the set of slots whose values are requested by the user
in the current turn and the values assigned to different slots in the service
based on the dialogue context. The values assigned to different slots are
represented as a list. For categorical slots, this list takes exactly one value,
and this value is listed in the schema. Arbitrary string values are supported
for non-categorical slots. We constrain these values to be substrings in the
user or the system utterance. For such slots, the list of values contains all
variations for the same value over the dialogue seen till the current turn (e.g,
"6 pm", "six in the evening", "evening at 6" are spoken variations for the same
value). The state tracker can output any of these variations. This decision was
made to avoid the additional complexity of obtaining the canonicalized version
of a value on the dialogue state tracker. In the real world, this
canonicalization can be done by the service being called or by a separate
module. For date slots, some of the dialogues contain a relative quantifier
(e.g, "today", "tomorrow", "next thursday" etc.). For these values, March 1st,
2019 has been treated as today's date for all dialogues.

### Single Domain Dataset

The single domain dataset includes dialogues involving interactions with a
single service, possibly over multiple intents. The overall statistics of
the train and dev sets are given below. The term *informable slots* refers to
the slots over which the user can specify a constraint. For example, slots like
*phone_number* are not informable.


|                                                   | Train  | Dev      |
| ------------------------------------------------- | :----: | :------: |
| No. of dialogues                                  | 5414   | 836      |
| No. of turns                                      | 82758  | 11928    |
| No. of tokens (lower-cased)                       | 808598 | 117510   |
| Average turns per dialogue                        | 15.286 | 14.268   |
| Average tokens per turn                           | 9.771  | 9.852    |
| Total unique tokens (lower-cased)                 | 16395  | 6804     |
| Total no. of slots                                | 205    | 134      |
| Total no. of informable slots                     | 147    | 94       |
| Total unique slot values (lower-cased)            | 7081   | 2418     |
| Total unique informable slot values (lower-cased) | 3847   | 1223     |
| Total domains                                     | 14     | 16       |
| Total services                                    | 25     | 17       |
| Total intents                                     | 35     | 28       |

The following table shows how the dialogues and services are distributed among
different domains for the train and dev sets. Please note that a few domains
like *Travel* and *Weather* are only present in the dev set. This is to test the
generalization of models on unseen domains. The test set will similarly have
some unseen domains which are neither present in the training nor in the dev
set.


| Domain      | # Dialogues <br> Train | # Services <br> Train | # Dialogues <br> Dev | # Services <br> Dev |
| :---------- | :---------: | :--------: | :--------------: | :-------------: |
| Alarm       | NA          | NA         | 37               | 1               |
| Banks       | 207         | 1          | 42               | 1               |
| Buses       | 310         | 2          | 44               | 1               |
| Calendar    | 169         | 1          | NA               | NA              |
| Events      | 788         | 2          | 73               | 1               |
| Flights     | 985         | 2          | 94               | 1               |
| Homes       | 268         | 1          | 81               | 1               |
| Hotels      | 458         | 3          | 56               | 2               |
| Media       | 266         | 1          | 46               | 1               |
| Movies      | 311         | 1          | 47               | 1               |
| Music       | 394         | 2          | 35               | 1               |
| RentalCars  | 215         | 2          | 39               | 1               |
| Restaurants | 367         | 1          | 73               | 1               |
| RideSharing | 119         | 2          | 45               | 1               |
| Services    | 557         | 3          | 44               | 1               |
| Travel      | NA          | NA         | 45               | 1               |
| Weather     | NA          | NA         | 35               | 1               |


### Multi Domain Dataset

Expected to be released by 19th July.

## Evaluation

The following metrics are defined for evaluation of dialogue state tracking. A
python script for computing these metrics will be released soon with additional
details. The joint goal accuracy will be used as the primary metric for ranking
submissions.

1.  **Active intent accuracy** - The fraction of user turns for which the active
    intent has been correctly predicted.
2.  **Slot tagging F1** - The macro-averaged F1 score for tagging slot values
    for non-categorical slots. This metric is optional to report in the final
    paper if participants decide not to use slot tagging.
3.  **Requested slots F1** - The macro-averaged F1 score for requested slots
    over the turns. For a turn, if there are no requested slots in both the
    ground truth and the prediction, that turn is skipped. The reported number
    is the average F1 score for all un-skipped user turns. This metric is
    optional to report in the final paper.
4.  **Average goal accuracy** - For each turn, participants must predict a
    single value for each slot present in the dialogue state. The slots which
    have a non-empty assignment in the ground truth dialogue state are only
    considered. This is the average accuracy of predicting the value of a slot
    correctly. A fuzzy matching based score is used for non-categorical slots.
    Note that we don't report per-slot metrics because of a large variety of
    slots in this dataset, and the set of slots present in the training, dev and
    test set are not identical.
5.  **Joint goal accuracy** - This is the average accuracy of predicting all
    slot assignments for a turn correctly. A fuzzy matching based score is used
    for non-categorical slots. This is the primary evaluation metric used for
    ranking submissions. More details to follow with the evaluation script.

## Rules

*   Participation is welcome from any team (academic, corporate, government
    etc.).
*   The identity of participants will not be made public by the organizers. It
    may be orally announced at the workshop chosen for communicating results.
    The participants may choose to identify their own team in the paper.
*   Participants are allowed to use any external datasets, resources or
    pre-trained models.
*   The developed systems should be feasible to use in live systems in terms of
    runtime.
*   Manual inspection of test set is not permitted. This is enforced on an
    honorary basis.
*   Participants may report results on either or both of the single domain and
    multi domain datasets.

## Acknowledgements

We thank Amir Fayazi, Maria Wang, Ulrich Rueckert and Jindong Chen for their
valuable suggestions and support in the formulation of this track and collection
of this dataset.
