## Virtual Attendants ##

The goal in the design of Virtual Attendants was to provide a way for workers and group managers to automate various workflows without having to create custom plugins.  Workers can now automate sophisticated business logic from their browser without any knowledge of programming software.

This may sound complicated, but taking the time to understand how Virtual Attendants work will give you some incredibly powerful tools to apply to countless situations.  For example:

* Generating notifications in response to specific kinds of email
* Setting due dates based on the sender's service level agreement (SLA)
* Sending auto-replies
* Quarantining spam
* Assigning incoming tickets to workers based on various criteria
* Organizing incoming work into buckets
* Relaying messages to workers' mobile phones

Virtual Attendants can even create tasks, write comments, automatically reply to customers, and set custom field values.  One of the most powerful aspects of Attendants is that you aren't limited to our built-in conditions and actions.  Through plugins, Attendants can even send SMS text messages to workers' mobile phones, or write messages to instant messengers and chat rooms.  While we've tried to eliminate the requirement of plugins for automation, combining plugins and Virtual Attendants will allow you to automate almost any kind of behavior.

Virtual Attendants monitor specific _events_ and react with pre-defined _behavior_.  Each _decision_ chooses the best _outcome_ and runs the associated _actions_.

### Events ###

An _event_ is triggered when something noteworthy occurs.

Examples: 

* A customer replies to a conversation.
* A worker replies to a customer.
* A conversation is moved to a new bucket.
* A conversation is marked closed.

### Behavior ###

A _behavior_ is a set of linked decisions that determine how to react to an event.

Examples:

* If new mail is received during the weekend, and the sender has a paid support plan, then relay the message to a list of worker mobile email addresses.
* When new mail is delivered to the Inbox, and it has a high probability of being spam, then move it to the "Spam" bucket.
* If a new order is received, send an SMS message to a list of worker mobile phones.
* If it's Friday, then decide if it's before or after 6pm.  If before 6pm, do nothing.  If after 6pm, send an automatic reply to the recipients that we've left the office for the weekend but will return at 8am on Monday, and give them emergency contact information.
* If a new message has the subject "send me a price list", then reply with a pre-defined message detailing the current prices.
* If a new message has the subject "unsubscribe", then set the "Do not contact" custom field on the sender address to "yes" and move the message to the "Cancels" bucket.

### Decisions ###

A _decision_ chooses the best outcome from several possible choices.  The result of a decision can be a series of actions, or another decision; or both.

Examples:

* Is today either Monday or Friday?
* Is it after 8am and before 6pm?
* Is this conversation in the Inbox bucket?
* Does the sender's organization have 'Software' checked in the 'Industries' custom field?

### Conditions ###

A _condition_ is the comparison of a single property to one or more values using an operator.  There are many possible operators depending on the type of field: equals, doesn't equal, contains phrase, is greater than, is any of, etc.  The result of a condition will always be either `true` or `false`.

Examples:

* "Time of Day" [is not] before 8am
* "Day of Week" [is any of] Saturday or Sunday
* "Message Subject" [starts with] "Receipt"

### Outcomes ###

The first _outcome_ to pass all of its conditions is selected by the decision.  The final outcome in a list of choices can have zero conditions in order to catch every other outcome.  If no outcomes match the behavior is terminated.

Examples:

* Decision: What day of week is it?
	* Outcome #1: Friday, Saturday, or Sunday
	* Outcome #2: Anything else
* Decision: What time of day is it?
	* Outcome #1: Before 8am
	* Outcome #2: Before 6pm
	* Outcome #3: After 6pm
* Decision: What bucket is the conversation in?
	* Outcome #1: Inbox
* Decision: Does the sender's first name begin with "J"?
	* Outcome #1: Yes
	* Outcome #2: No
* Decision: What is the value of the service level custom field on sender's organization?
	* Outcome #1: V.I.P.
	* Outcome #2: Paid support
	* Outcome #3: Anything else (Regular support)

### Actions ###

An _action_ defines a set of things to do.

Examples:

* Write a comment with the text "Automatically added to billing."
* Create task with the title "Follow-up in 3 days" and add a list of workers as watchers.
* Send a reply to the recipients using a template with placeholders for first name and company name.
* Close the conversation.
* Send an SMS message to a list of mobile phones.
