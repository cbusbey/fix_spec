fix_spec
========

Build and Inspect FIX Messages

### DataDictionary

FIXSpec works best when a DataDictionary is provided.  With a DataDictionary loaded, you can inspect a message with named tags, enumeration, and type specific tag values.

The DataDictionary is globally set:

```ruby
FIXSpec::data_dictionary = quickfix.DataDictionary.new "config/FIX42.xml"
```

### Exclusion

When checking for fix message equality, you may wish to ignore some common fields that are mostly session level.  For example, at an application level, BodyLength and CheckSum can be assumed to be set correctly. Tag exclusion is configured globally via JsonSpec:

```ruby
JsonSpec.configure do
  exclude_keys "BodyLength", "CheckSum", "MsgSeqNum"
end
```

Cucumber
--------

fix_spec provides Cucumber steps that utilize its RSpec matchers.

In order to us ethe Cucumber steps, in your `env.rb` you must:

```ruby
require "fix_spec/cucumber"
```

### "Should" Assertions

In order to test the contents of a FIX message, you will need to define a `last_fix` method.  This method will be called by fix_spec to grab the FIX message to test. For example, suppose a step aquires a fix message and assigns it to `@my_fix_message`.  In your `env.rb` you could then have

```ruby
def last_fix
  @my_fix_message
end
```

See `features/support/env.rb` and `features/step_definitions/steps.rb` for a very simple implementation.

Now you can use fix_spec steps in your features:

```cucumber
Feature: New Order

Background:

Given some order message

Scenario: A Market Order is valid
Then the FIX message type should be "NewOrderSingle"
Then the FIX should have tag "OrderQty"
And the FIX message should not have tag "Price"
And the FIX at "SenderCompID" should be "MY_SENDER"

And the FIX messsage should have the following:
|SenderCompID | "MY_SENDER" |
|TargetCompID | "MY_TARGET" |
|OrderQty     | 123         |
|OrdType      | "MARKET"    |

And the FIX message should be:
"""
{
  "BeginString":"FIX.4.2",
  "BodyLength":81,
  "MsgType":"NewOrderSingle",
  "SenderCompID":"MY_SENDER",
  "TargetCompID":"MY_TARGET",
  "OrdType": "MARKET",
  "OrderQty": 123,
  "CheckSum":"083"
}
"""

And the FIX message should be:
"""
8=FIX.4.29-8135=D49=MY_SENDER56=MY_TARGET40=138=12310=083
"""
```

The background step isn't provided by fix_spec.  The remaining steps fix_spec provides. See `features/` for more examples.


### Building FIX Messages

In order to use the Cucumber steps for building FIX messages, in your `env.rb` you must:

```ruby
require "fix_spec/builder"
```
Now you can use fix_spec builder steps in your features:

```cucumber
Feature: Order Adapter accepts Orders

Scenario: It accepts Market Orders

Given I create a FIX.4.2 message of type "NewOrderSingle" 
And I set the FIX message at "SenderCompID" to "MY_SENDER"
And I set the FIX message at "TargetCompID" to "MY_TARGET"
And I set the FIX message at "OrdType" to "MARKET"
And I set the FIX message at "OrderQty" to 123
Then I send the message
```

The built FIX message can be accessed through the `message` function in the ```FIXSpec::Builder``` module.


Setup
-----

    bundle install

Test
----

    bundle exec rspec
    env JAVA_OPTS=-XX:MaxPermSize=2048m bundle exec rake cucumber
