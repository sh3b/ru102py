Client Protocol
The Redis Protocol

The Redis protocol was designed with simplicity in mind. For this reason, it's text-based and human-readable. You can read all about the protocol on the redis.io documentation page.

Here's a small example. If you run the command:

lpush test hello

It will be encoded as follows (Note that the protocol is delimited by newlines, which consist of a carriage return (\r) followed by a line feed (\n):

*3\r\n
$5\r\n
lpush\r\n
$4\r\n
test\r\n
$5\r\n
Hello\r\n

Recall that the command consists of three strings: "lpush", "test" and "hello".

So the first line "*3" indicates that we're sending Redis an array of three elements. The "*" indicates the array type, and the "3" indicates length. The three strings will follow.

The first element of the command we're running is a 5-character string. So we pass in a "$" to signify the string type and follow that with a 5 to indicate the size of the string. The next line is the string itself, delimited by a newline: "lpush\r\n".

The next two strings follow the same pattern: ":$" to indicate a string, then a length, then a CRLF.

This command has an integer reply of 1. In the Redis protocol, that reply looks like this:

:1\r\n

Here, the ":" character indicates that the reply is an integer. This is followed by an integer. The line is then terminated with a CRLF.
Viewing the Redis Protocol Live

If you're on a Linux machine, you can observe the Redis protocol in action using ngrep:

ngrep -W byline -d lo0 -t '' 'port 6379'

If you can't use ngrep, Wireshark also works.
