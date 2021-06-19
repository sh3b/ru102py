# String Encodings and redis-py

In the last section, we looked at code that passed decode_responses=True to the Redis() initializer. And we said that doing so caused redis-py to decode Redis strings using the UTF-8 encoding.

If you're unfamiliar with string encodings, you might be asking: what does that mean?

This section is for you.

**Note:** Already know about string encodings? Feel free to skip this and return if you have questions later!

## Bytes versus strings

Redis strings are binary sequences. When you store a binary sequence, you typically do so with an encoding. Here are a few common encodings:

*   UTF-8
*   WAV
*   JPEG

You can store data in any of these encodings -- and many more -- as Redis strings.

But what happens when you later retrieve the strings using redis-py? Do you get the data back in the original encoding?

No! That’s because neither Redis nor redis-py assume anything about the encoding of Redis strings. What you get back by default are binary sequences, which Python represents as "bytes" objects.

You must then decode these objects into Python strings using an encoding.

## The decode_responses parameter

redis-py’s Redis() initializer takes a decode_responses parameter that defaults to False. If you don’t provide an argument for this parameter, redis-py behaves as described previously, returning bytes objects for Redis commands like GET that return binary-safe strings.

However, if you pass the argument True for this parameter, redis-py will decode all binary-safe strings using an encoding. The default is UTF-8, and you can specify a different encoding with the encoding parameter.

This is useful if you store string data in Redis with a single encoding. The example project for this course uses UTF-8, so all the Redis clients in the project use decode_responses=True to decode strings with the UTF-8 encoding.

## When should you use decode_responses=False?

When should you use the default behavior of redis-py and work with bytes objects instead of decoding responses?

You should do so if you store data in Redis in mixed encodings. In this case, you have a couple of options:

*   Create multiple Redis client instances, and scope each instance to a particular encoding like JPEG or UTF-8
*   Use the default behavior of redis-py and decode bytes objects returned from Redis commands manually

## Next up…

OK, you should now have a better grasp on why string encodings matter when using redis-py.

Still unsure? Read up on string encodings like UTF-8, how the bytes and string objects work in Python, and how to convert between the two.

Here are a couple of websites to get you started:

*   [A "painless" introduction to string encodings in Python](https://realpython.com/python-encodings-guide/)
*   [A super deep look at UTF-8](https://utf8everywhere.org/)

Next, we’re going to look closer at how to run some Redis client basics.

----------------------------------------------------------------

