
# Creating triggers and rules

OpenWhisk triggers and rules bring event-driven capabilities to the platform. Events from external and internal event sources are channeled through a trigger, and rules allow your actions to react to these events.

## Triggers

Triggers are a named channel for a class of events. The following are examples of triggers:
- A trigger of location update events.
- A trigger of document uploads to a website.
- A trigger of incoming emails.

Triggers can be *fired* (activated) by using a dictionary of key-value pairs. Sometimes this dictionary is referred to as the *event*. As with actions, each firing of a trigger results in an activation ID.

Triggers can be explicitly fired by a user or fired on behalf of a user by an external event source.
A *feed* is a convenient way to configure an external event source to
fire trigger events that can be consumed by OpenWhisk. Examples of feeds include the following:
- Cloudant data change feed that fires a trigger event each time a document in a database is added or modified.
- A Git feed that fires a trigger event for every commit to a Git repository.

## Rules

A rule associates one trigger with one action, with every firing of the trigger causing the corresponding action to be invoked with the trigger event as input.

With the appropriate set of rules, it's possible for a single trigger event to
invoke multiple actions, or for an action to be invoked as a response to events
from multiple triggers.

For example, consider a system with the following actions:
- `classifyImage` action that detects the objects in an image and classifies them.
- `thumbnailImage` action that creates a thumbnail version of an image.

Also, suppose that there are two event sources that are firing the following triggers:
- `newTweet` trigger that is fired when a new tweet is posted.
- `imageUpload` trigger that is fired when an image is uploaded to a website.

You can set up rules so that a single trigger event invokes multiple actions, and have multiple triggers invoke the same action:
- `newTweet -> classifyImage` rule.
- `imageUpload -> classifyImage` rule.
- `imageUpload -> thumbnailImage` rule.

The three rules establish the following behavior: images in both tweets and uploaded images are classified, uploaded images are classified, and a thumbnail version is generated. 

## Creating and firing triggers

Triggers can be fired when certain events occur, or can be fired manually.

As an example, create a trigger to send user location updates, and manually fire the trigger.

1. Enter the following command to create the trigger:
 
  ```
  $ wsk trigger create locationUpdate
  ```
 
  ```
  ok: created trigger locationUpdate
  ```

2. Check that you created the trigger by listing the set of triggers.

  ```
  $ wsk trigger list
  ```
 
  ```
  triggers
  /someNamespace/locationUpdate                            private
  ```

  So far you've created a named "channel" to which events can be fired.

3. Next, fire a trigger event by specifying the trigger name and parameters:

  ```
  $ wsk trigger fire locationUpdate --param name "Donald" --param place "Washington, D.C."
  ```

  ```
  ok: triggered locationUpdate with id fa495d1223a2408b999c3e0ca73b2677
  ```

  A trigger that is fired without an accompanying rule to match against has no visible effect.

  Triggers may not be created inside a package, they must be created directly under a namespace.

## Using rules to associate triggers and actions

Rules are used to associate a trigger with an action. Each time a trigger event is fired, the action is invoked with the event parameters.

As an example, create a rule that calls the hello action whenever a location update is posted. 

1. Create a 'hello.js' file with the action code we will use:
  ```
  function main(params) {
     return {payload:  'Hello, ' + params.name + ' from ' + params.place};
  }
  ```

2. Make sure that the trigger and action exist.
  ```
  $ wsk trigger update locationUpdate
  ```
  
  ```
  $ wsk action update hello hello.js
  ```

3. Create and enable the rule. The three parameters are the name of the rule, the trigger, and the action.
  ```
  $ wsk rule create --enable myRule locationUpdate hello
  ```

4. Fire the locationUpdate trigger. Each time you fire an event, the hello action is called with the event parameters.
  ```
  $ wsk trigger fire locationUpdate --param name "Donald" --param place "Washington, D.C."
  ```
  
  ```
  ok: triggered locationUpdate with id d5583d8e2d754b518a9fe6914e6ffb1e
  ```

5. Verify that the action was invoked by checking the most recent activation.
  ```
  $ wsk activation list --limit 1 hello
  ```
  
  ```
  activations
  9c98a083b924426d8b26b5f41c5ebc0d             hello
  ```
  
  ```
  $ wsk activation result 9c98a083b924426d8b26b5f41c5ebc0d
  ```
  ```
  {
     "payload": "Hello, Donald from Washington, D.C."
  }
  ```

  You see that the hello action received the event payload and returned the expected string.

  You can create multiple rules that associate the same trigger with different actions.

  The trigger and action that make a rule must be in the same namespace and cannot belong to package.
  If you want to use an action that belongs to a package, you can copy the action into your namespace, for example `wsk action create echo --copy /whisk.system/samples/echo`.
