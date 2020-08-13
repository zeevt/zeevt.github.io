There was a customer who wanted us to do a little project.

The boss who was in charge there thought it should have been a month's worth of work.

We thought their requirements are impossible to implement,
and tried quite hard to get them to change the requirements to something that could be implemented.

At first they seemingly thought we were trying to upsell a completely unnecessary requirements gathering phase.

Then they agreed to do time bounded requirements gathering + design,
and we produced spec, design and an estimate
and they thought we were way off on our cost estimate and that's the end of that project.

What was the sticking point?

They wanted a system that was a web UI and REST API for managing a kind of schema-full document store database on top of postgresql.

Of course, that's not what they called it.

You could create a "table" (they didn't call it that), let's say "employees" with columns (they didn't call it that) "id" and "alias".

In this example "alias" is a compound field (they didn't call it that) composed of "firstName", "middleName" and "LastName".

The cool feature is that "middleName" has a toggle for "allows multiple values" and
"alias" also has a toggle for "allows multiple values" and the toggles
can be switched in both directions when there is data in the database, not only at table creation time.

So a user record/document can be something like this:

```json
{
  "id": 123,
  "alias": [
    {"firstName": "Felipe VI", "lastName": "de España"},
    {"firstName": "Felipe", "middleName": ["Juan", "Pablo", "Alfonso"], "lastName": "de Todos los Santos"},
  ]
}
```

and the schema changes should be atomic, transactional, online, and performant,
even if the table already has millions of existing documents.

There were a lot of other features, but those were simpler.

While I was working on it, I spent all my time thinking about the technical details,
how could I implement their requirements and how could I
better explain why their requirements are hard to implement.

I was having a huge problem convincing them of this and getting answers for my questions
about what was and wasn't allowed within the system.

On the last day, when we needed their answers for the last questions to finish the spec
and deliver it, their PM who was assigned to the project, who was my single point of contact the entire time,
was late replying by email and we simply didn't have any more time.

So I called him on the phone.

Now, we have met face to face before, in the beginning of the project.

During that phone call I realized what the problem had been for weeks.

I thought the problem the whole time was solely that he was non-technical, their requirements
caused many technical problems, I tried hard to explain the technical problems and present
alternatives using both technical jargon and non-technical explanations, and I was getting nowhere.

Turns out, the most serious problem was that his English was poor and he just couldn't read my spec or my emails very well.

The face to face meeting was in Hebrew.

Had he simply asked us to write the spec and do all the correspondence in Hebrew, we would have done that at no additional cost.

Instead, he was too embarrassed to speak up, I was tearing my hair out,
and we couldn't simplify the requirements enough to make the cost close to what they envisioned.
