---
layout: post
title: "Stop a while loop by pressing a key"
date: 2018-04-11
---
For several applications, you might want to stop a while by pressing a key. It can be useful for example to prevent `Ctrl-C` a script from happening (which can be problematic for closing opened files, ...). One possibility might be to stop a loop just by pressing a key. Sadly, there are no direct way to do that in python.

# Solution with try/except

One solution could be to use exceptions and to include your whole program into a try ([snippet found here](https://bytes.com/topic/python/answers/701570-there-simple-way-exit-while-loop-keystroke)):

```
from time import sleep

def do_things():
  try:
    while 1:
      print "in loop"
      sleep(1)
  except KeyboardInterrupt:
      print "loop is done"

if __name__ == "__main__":
  do_things()

```
The program will try to run the `while 1:` until it sees a keyboard interruption (in our case, it's a `Ctrl-C`). In that case, instead of quitting the script, it will run the `except KeyboardInterrupt:` part. Despite that this solution works, it needs `Ctrl-C` to stop the loop and force yourself to put your whole script into a `try`.

# Solution with threads

Another solution would be to use the `raw_input` from python which awaits te user to press `Enter`. In order to prevent the waiting part, a solution found [here](https://stackoverflow.com/questions/13180941/how-to-kill-a-while-loop-with-a-keystroke) could be to use a different thread for the key pressing:

```
import thread
from time import sleep

def input_thread(continue_list):
  raw_input()
  continue_list[0] = False

def do_things():
  continue_list = [True]
  thread.start_new_thread(input_thread, (continue_list,))
  while continue_list[0]:
    print "in loop"
    sleep(1)
  print "loop is done"

if __name__ == "__main__":
  do_things()

```

In that case, a new thread will be started (called `input_thread`) and will be given the variable `continue_list` which is a ... list! Therefore, the original `continue_list` and the threaded one will both be pointing at the same memory adress `continue_list[0]` (initialized at `True`). The loop from `do_things` will keep running until `continue_list[0]` is modified to `False`. This can only happen in the threaded part, which will be blocked (waiting) at the `raw_input()` until the user press `Enter`. In that case, `continue_list[0]` will be changed (in the threaded `input_thread` and in `do_things()`) and will exit the loop. Despite that only the `Enter` key works, it found this solution rather nice and elegant compared to the previous one. In order to take another key as input, you might need to load some specific librairy for that.

*Note*: The previous link also includes a solution based on opencv. Despite that it works nicely and with any desired key, it still requires the use of the opencv librairy, which can be heavy for such a small task.
