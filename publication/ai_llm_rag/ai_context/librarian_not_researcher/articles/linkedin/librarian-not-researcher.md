Your AI should behave like a librarian, not a researcher

I watched a team “give autonomy” to an assistant by asking it to research a problem end-to-end.

It looked impressive right up until the first incident review:

- nobody could explain why it picked those sources
- nobody could predict how long it would keep trying
- nobody could bound the cost envelope without weakening the prompt

The uncomfortable truth is that “research” is a decision-making posture.
And decision-making is the thing you most need to constrain.

The better mental model is a library:

- the model is the person asking for information
- the orchestration layer is the librarian
- tool definitions are catalogue cards
- stores and pipelines decide what exists; the librarian decides what is findable

Once you adopt that posture, architecture choices change.
You stop optimising for improvisation and start building a governable control surface:
explicit sources, structured access, bounded behaviour, auditable decisions.

📌 Autonomy in production is not “trusting the model more”. It is trusting the system you built around it.

Question: where in your system have you accidentally asked the model to be the decision-maker?

#AI #AIEngineering #Architecture #ClaudeCode #Trae
