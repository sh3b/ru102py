
Hands-on

Check out the MONITOR documentation https://redis.io/commands/monitor . Then run MONITOR from the Redis CLI while you execute the RediSolar application test suite.

If you're using the virtual lab, you'll need to run MONITOR in the background, since you only have a single terminal window to work with. You can also start MONITOR in the background like so:

$ redis-cli monitor > commands.txt &

Next, run the test suite:

$ make test

Now check the file commands.txt:

$ cat commands.txt

You should see all of the commands sent to Redis during the test suite.

You can stop the MONITOR process by bringing it to the foreground:

$ fg

And then typing CTRL-C.
