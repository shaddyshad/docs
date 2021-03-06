# Securing & Transforming data

DevLess comes with built-in tooling for securing and transforming data. For example, you can allow only admins to write to your "products" table or normalize all emails to be lowercase.

## Securing data

By default, almost all endpoints in DevLess are public. This is good for enabling easy development, but bad for production.

Securing data in DevLess can be done on two levels. One , based on endpoint access rules, which applies to the entire service. You can also apply more fine-grained access by using the _rules engine_. The _rules_ engine allows you to apply logic when reading from or writing to the database. This includes the ability to do fine-grained access checks.

### Endpoint Access Rules

You can change the endpoint access rules going to your DevLess admin panel, and clicking on the "privacy" menu item on the left. Here you can apply access rules for querying, writing etc. on a per-service basis. By default, almost all are public.

* `PUBLIC` access means that anyone with an API key can use the resource. 
* `AUTHENTICATE` access means that **any** logged in user can use the resource. 
* `PRIVATE` access that no-one can access the resource. This can be overridden in the rules engine. For production ready systems, you probably want this on all resources.

### Securing using the rules engine

DevLess comes with a rules engine. The rules engine can be used to apply table-specific grants for reading or writing. You can access the rules which applies to your service in the DevLess admin UI. Under services, select your service and then select the "rules" tab.

Here you will see something like this \(you might have to scroll in the editor\):

```php
 -> beforeQuerying()
 -> beforeUpdating()
 -> beforeDeleting()
 -> beforeCreating()

 -> onQuery()
 -> onUpdate()
 -> onDelete()
 -> onCreate()

 -> onAnyRequest()

 -> afterQuerying()
 -> afterUpdating()
 -> afterDeleting()
 -> afterCreating()
```

Lets take a look at how to allow only admins to create entries in the `people` table. Start with setting the endpoint access rule for creating data to `PRIVATE`. We can now grant admins only access to creating data in the `people` table:

```php
-> beforeCreating()->onTable('people')->grantOnlyAdminAccess()
```

We can also make viewing the people table public:

```php
-> beforeQuerying()->onTable('people')->allowExternalAccess()
```

### Video tutorial

For a more in-depth and hands-on walk-through of securing data, there is [a video tutorial](https://www.youtube.com/watch?v=SOlXNSPFmOg).

## Transforming data

The rules engine can also transform data, both before writing to the database and after reading from it. This is done by using the same hooks as when securing data.

### Before storing

For example, you can normalize all emails to be lowercase by using the beforeCreating hook.

```php
->beforeCreating()->onTable("people")->convertToLowerCase($input_name)->storeAs($input_name)`
```

What happens here is that for the `people` table convert all inputs for the field `name` to be lower-cased with the `convertToLowerCase` method. We then overwrite the `$input_name` variable with `storeAs`. The `$input_name` is the variable that will be stored in the database in the `name` field.

[This video](https://www.youtube.com/watch?v=z6CXQhcQz6I) goes more into depth on how you can transform data before storing.

### Before returning data to the client

We can also manipulate data before we send it back to the client. There are three main methods for doing this.

For modifying the response message, use the `mutateResponseMessage(new_message)` method. The message is used for notifying the client about what happened using a textual format. You can use it to e.g. give a more context aware message:

```php
-> afterCreating()->onTable("people")->mutateResponseMessage("Contact added")`
```

For modifying the payload, use the `mutateResponsePayload(payload_to_add)`. This can be used to add additional information for your clients.

E.g. to add the timestamp at which the server returned the payload, we can do this:

```php
->afterQuering()->onTable("people")->getTimeStamp()->storeAs($timestamp)->mutateResponsePayload(["timestamp"=>$timestamp])
```

We can also mutate the status code. This is for **advanced users only**. Modifying this will impact how SDKs and clients interpret the response, so proceed with caution. Use the `mutateStatusCode` method to change the status code.

[This video](https://youtu.be/a2ScbtehNeE) goes more into depth on how you can use data manipulation to affect what the clients see.

### Flow control

The rules engine supports conditional flow, similar to the `if`/`else if`/`else` conditionals in programming languages. The functions are named `whenever`, `elseWhenever` and `otherwise`.

The flow control functions plays nicely together with the [`assertIts`](https://github.com/devless/devless-docs-1-3-0/tree/949b00258810c377469907e7bc8021ecb2d4412d/assertion-list.md) family of functions. Together, these functions allows you to take different actions depending on the user input.

For example, we can show different messages depending on the email domain:

```php
afterCreating()->onTable("people")->whenever(assertIts::endsWith($input_email, "gmail.com"))->mutateResponseMessage("Welcome gmail user")
```

For a deeper dive into working with flow control in rules, see [this video](https://www.youtube.com/watch?v=Mwurl21niSw)

