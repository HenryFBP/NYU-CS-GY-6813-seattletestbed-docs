# Repy Programming Guide

## Introduction and Purpose

This document describes the current, production version of the narrow 
API available to Repy programs, **Repy V1**. (The upcoming API, Repy V2, 
is documented here: https://github.com/SeattleTestbed/docs/blob/master/Programming/RepyV2API.md)

This document includes all of the calls that are available to a Repy 
program and the use and meaning of these calls.   For items built into 
the Python programming language (like list operations, etc.) see the 
appropriate Python documentation.   

## Detailed description

### Shared variables

#### mycontext
  Mycontext is a dictionary that is shared between all of the threads in 
  your program. The intent is that mycontext will be used in the place of 
  global variables. See the tutorial for examples using mycontext.

#### callfunc
  Your program may choose to have different functionality that executes at 
  different times.   The callfunc variable is set to values that indicate 
  what the state of execution is.   Callfunc is set to either "initialize" 
  and "exit".   These are executed by running the program as a script and 
  setting the variable callfunc to either the string "initialize" or the 
  string "exit".   The "initialize" is the first time that your program 
  executes.   If initialize finishes and there are no pending threads or 
  callbacks, the program is called with "exit" to allow it a chance to 
  clean up.

#### callargs
  Callargs is a list of strings that contain the arguments your program 
  was called with.   If there are no arguments, callargs contains an 
  empty list.

### API functions
----
 
#### gethostbyname_ex(name)
----
(from the Python documentation)   Translate a host name to IPv4 address format, extended interface. Return a triple (hostname, aliaslist, ipaddrlist) where hostname is the primary host name responding to the given ip_address, aliaslist is a (possibly empty) list of alternative host names for the same address, and ipaddrlist is a list of IPv4 addresses for the same interface on the same host (often but not always a single address). gethostbyname_ex() does not support IPv6 name resolution, and getaddrinfo() should be used instead for IPv4/v6 dual stack support. 

 * Doc string
```python
"""
  <Purpose>
    Provides information about a hostname.   Calls socket.gethostbyname_ex()

  <Arguments>
    name:
       The host name to get information about

  <Exceptions>
    As from socket.gethostbyname_ex()

  <Side Effects>
    None.

  <Returns>
    A tuple containing (hostname, aliaslist, ipaddrlist).   See the 
    python docs for socket.gethostbyname_ex()
"""
```

#### getmyip()
----
Returns the localhost's "Internet facing" IP address.   It may raise an exception on hosts that are not connected to the Internet.


 * Doc string
```python
"""
  <Purpose>
    Provides the external IP of this computer.   Does some clever trickery.

  <Arguments>
    None

  <Exceptions>
    Exception if the host is not connected to the Internet or has name resolution problems

  <Side Effects>
    None.

  <Returns>
    The localhost's IP address
"""
```

#### recvmess(localip, localport, function)
----

Registers a function as a callback for incoming UDP messages.  If another message arrives before the first is processed, a second thread will be started. If this function is called multiple times on the same ip and port, the previous function is replaced.   When an incoming UDP message is delivered on this IP and port, the function is called.   If there is no free thread / event, the function is not called until a thread / event is available.   Also, if there is no current function registered by recvmess or waitforconn, an thread / event is consumed to check for incoming network traffic.   This function returns a commhandle that can be used by stopcomm to deregister the handler.   The maximum datagram size is 4096 bytes in Repy 0.1r, but later revisions will remove this restriction.

 * Doc string
```python
"""
  <Purpose>
    Registers a function as a callback for incoming messages

  <Arguments>
    localip:
      The local IP or hostname to register the handler on
    localport:
      The port to listen on
    function:
      The function that messages should be delivered to.   It should expect
      the following arguments: (remoteIP, remoteport, message, commhandle)

  <Exceptions>
    None.

  <Side Effects>
    Registers a callback.

  <Returns>
    The commhandle for this callback.
"""
```

#### sendmess(desthost, destport, message, localip=None, localport=None)
----
Sends a UDP message to a destination host / port.   By default the system selects a localip and port for the outgoing message, but this can be overridden by passing in arguments that specify which port and IP to use.   If either the localip or localport are used, both must be.   Returns the number of bytes sent.

 * Doc string
```python
"""
  <Purpose>
    Send a message to a host / port

  <Arguments>
    desthost:
      The host to send a message to
    destport:
      The port to send the message to
    message:
      The message to send
    localhost (optional):
      The local IP to send the message from 
    localport (optional):
      The local port to send the message from 

    Note: if you specify localhost, you must specify localport

  <Exceptions>
    socket.error when communication errors happen

  <Side Effects>
    None.

  <Returns>
    The number of bytes sent on success
"""
```

#### openconn(desthost, destport, localip=None, localport=0, timeout = 5)
----
Open a TCP connection to a remote computer, returning a socket object.   Optionally a localip and localport to connect from can be specified and if one is specified both must be.   There is a timeout value that can be set to limit the amount of time the system will wait for a response before abandoning the attempt to connect.

 * Doc string
```python
"""
  <Purpose>
    Opens a connection, returning a socket-like object

  <Arguments>
    desthost:
      The host to open communications with
    destport:
      The port to use for communication
    localip (optional):
      The local ip to use for the communication
    localport (optional):
      The local port to use for communication
    timeout (optional):
      The maximum amount of time to wait to connect

    Note: if you specify localip, you must specify localport

  <Exceptions>
    As from socket.connect, etc.

  <Side Effects>
    None.

  <Returns>
    A socket-like object that can be used for communication.   Use send, 
    recv, and close just like you would an actual socket object in python.
"""
```


#### waitforconn(localip, localport, function)
----
Register a function to be called whenever a TCP connection is made to a localip and localport.   Multiple instances of the callback may execute concurrently if there are multiple incoming connections.   As with recvmess, an event / thread is consumed to start a network listener if no previous function was registered for either recvmess or waitforconn.   If a connection is established but no free events / threads remain, the function is not called until one is available.   Returns a handle that can be used by stopcomm to deregister the function.

 * Doc string
```python
"""
  <Purpose>
    Waits for a connection to a port.   Calls function with a socket-like 
    object if it succeeds.

  <Arguments>
    localip:
      The local IP to listen on
    localport:
      The local port to bind to
    function:
      The function to call.   It should take five arguments:
      (remoteip, remoteport, socketlikeobj, thiscommhandle, listencommhandle)
      If your function has an uncaught exception, the socket-like object it
      is using will be closed.
       
  <Exceptions>
    None.

  <Side Effects>
    Starts a handler that listens for connections.

  <Returns>
    A handle to the listener.   This can be used to stop listening
"""
```



#### stopcomm(commhandle)
----

Stops communication on a commhandle.   In the case of a UDP message based commhandle or a TCP communication listener commhandle, this deregisters the function and if there are no remaining functions registers, stops the event / thread that checks for network traffic.   This can also be called on the commhandle that corresponds to an open TCP connection and in that case it behaves identically to calling close for the connection.

 * Doc string
```python
"""
  <Purpose>
    Deregister a callback for a commhandle.   This works for both message and
    connection based callbacks.

  <Arguments>
    commhandle:
      A commhandle as returned by recvmess or waitforconn.

  <Exceptions>
    None.

  <Side Effects>
    This has an undefined effect on a socket-like object if it is currently
    in use.

  <Returns>
    Returns True if commhandle was successfully closed, False if the handle
    cannot be closed (i.e. it was already closed).
"""
```


#### open(filename, mode='r')
----

(some of this came from the Python docs)
Open a file, returning an object of the file type. If the file cannot be opened, IOError is raised.

Filenames may only be in the current directory and contain lower-case letters, numbers, the hyphen, underscore, and period character, and may not start with '.'.   There is no concept of a directory or a folder in repy.  

The most commonly-used values of mode are 'r' for reading, 'w' for writing (truncating the file if it already exists), and 'a' for appending. If mode is omitted, it defaults to 'r'.  Modes 'r+', 'w+' and 'a+' open the file for updating (note that 'w+' truncates the file).   The mode must begin with 'r', 'w' or 'a'.

 * Doc string
```python
"""
  <Purpose>
    Allows the user program to open a file safely.   This function is meant
    to resemble the builtin "open"

  <Arguments>
    filename:
      The file that should be operated on
    mode:
      The mode (see open)

  <Exceptions>
    As with open, this may raise a number of errors

  <Side Effects>
    Opens a file on disk, using a file descriptor.   When opened with "w"
    it will truncate the existing file.

  <Returns>
    A file-like object 
"""
```

#### file.close()
----

Close the file. A closed file cannot be read or written any more. Any operation which requires that the file be open will raise a ValueError after the file has been closed. Calling close() more than once is allowed.

 * Doc string
```python
"""
  <Purpose>
    Allows the user program to close a file.  This function is meant
    to resemble the builtin "file.close".

  <Arguments>
    None.

  <Exceptions>
    None.

  <Side Effects>
    Closes a file descriptor previously opened for writing.

  <Returns>
    Nothing.
"""
```

#### file.flush()
----

Flush the internal buffer, like C stdio's fflush().

 * Doc string
```python
"""
  <Purpose>
    Allows the user program to flush a file buffer.  This function is meant
    to resemble the builtin "file.flush".

  <Arguments>
    None.

  <Exceptions>
    ValueError if the file is closed.

  <Side Effects>
    Flushes a file previously opened for writing.

  <Returns>
    Nothing.
"""
```

#### file.next()
----

If the file pointer is at the end of the file, throws StopIteration. Otherwise, reads the next line from the file and returns it. Equivalent to the following:

```python
  def next(self):
    line = self.readline()
    if line == "":
      raise StopIteration()
    return line
```

 * Doc string
```python
"""
  <Purpose>
    Allows a user program to iterate over the lines of a file.

  <Arguments>
    None.

  <Exceptions>
    StopIteration if EOF is reached.
    ValueError if the file is closed.

  <Side Effects>
    Advances the file pointer to the next line (unless it is already at
    EOF).

  <Returns>
    The contents of the next line.
"""
```

#### file.read(size)
----

Reads up to size bytes of the rest of the (open) file, returning what is read. If size is ommitted, the file is read to EOF.

 * Doc string
```python
"""
  <Purpose>
    Reads from a file handle.

  <Arguments>
    size (optional) - Specify a maximum number of bytes to read from the
                      file.

  <Exceptions>
    ValueError if the file is closed.

  <Side Effects>
    Advances the file pointer.

  <Returns>
    The data that was read.
"""
```

#### file.readline(size)
----

Reads a whole line from the file. If size is not ommitted, the number of bytes read is limited to size. Trailing newlines are included in the resulting line. The empty string is returned if EOF is hit immediately.

 * Doc string
```python
"""
  <Purpose>
    Reads a line from a file.

  <Arguments>
    size (optional) - Specifies a maximum bound to the number of bytes
                      to read from the file.

  <Exceptions>
    ValueError if the file is closed.

  <Side Effects>
    Advances the file pointer.

  <Returns>
    The data read (a string).
"""
```

#### file.readlines(size)
----

Reads multiple lines from the file. If size is not ommitted and positive, stops reading when it has read at least size bytes.

 * Doc string
```python
"""
  <Purpose>
    Reads many lines from a file.

  <Arguments>
    size (optional) - Specifies an approximate bound to the number of bytes
                      to read from the file.

  <Exceptions>
    ValueError if the file is closed.

  <Side Effects>
    Advances the file pointer.

  <Returns>
    A list of lines read from the file. Lines are seperated by newlines, but
    these newlines are not stripped from the lines in the resulting list.
"""
```

#### file.seek(offset, whence=0)
----

Set the file???s current position, like stdio???s fseek().

 * Doc string
```python
"""
  <Purpose>
    Set the file's current position.

  <Arguments>
    offset - Specifies the number of bytes to seek.

    whence (optional) - Specifies how the seeking is to be done. Defaults to 0 (absolute
    file positioning); other values are 1 (seek relative to the current position) and 2
    (seek relative to the file's end). For example, f.seek(2, 1) advances the position by
    two and f.seek(-3, 2) sets the position to the third to last.

  <Exceptions>
    ValueError if the file is closed.

  <Side Effects>
    The file's current position is set.

  <Returns>
    Nothing.
"""
```

#### file.write(data)
----

Write some data to a file.

 * Doc string
```python
"""
  <Purpose>
    Allows the user program to write data to a file.

  <Arguments>
    Data to write (a string).

  <Exceptions>
    ValueError if the file is closed.
    IOError if the disk is out of space or some low level IO error occurs.

  <Side Effects>
    Writes some data after the current filepointer position, advancing it.

  <Returns>
    Nothing.
"""
```

#### file.writelines(lines)
----

Writes a sequence of strings to the file.

 * Doc string
```python
"""
  <Purpose>
    Writes a sequence of strings to a file. (Unlike the name might suggest,
    does not do any mucking about with adding or mangling newlines.)
    Equivalent to looping through the elements of the sequence and
    ```file.write()```ing them individually.

  <Arguments>
    lines - The sequence of strings to write to the file.

  <Exceptions>
    ValueError if the file is closed.
    TypeError if the ```lines``` argument isn't iterable.

  <Side Effects>
    Writes some data to the file.

  <Returns>
    Nothing.
"""
```

#### listdir()
----


Returns a list of file names for the files in the vessel.   

 * Doc string
```python
"""
  <Purpose>
    Allows the user program to get a list of files in their area.

  <Arguments>
    None

  <Exceptions>
    This probably shouldn't raise any errors / exceptions so long as the
    node manager isn't buggy.

  <Side Effects>
    None

  <Returns>
    A list of strings (file names)
"""
```


#### removefile(filename)
----

Deletes a file in the vessel.   If the file does not exist, an exception is raised.

 * Doc string
```python
"""
  <Purpose>
    Allows the user program to remove a file in their area.

  <Arguments>
    filename: the name of the file to remove.   It must not contain
    characters other than 'a-zA-Z0-9.-_' and cannot be '.' or '..'

  <Exceptions>
    An exception is raised if the file does not exist

  <Side Effects>
    None

  <Returns>
    None
"""
```

#### exitall()
----

Terminates the program immediately.   The program will not execute the "exit" callfunc or finally blocks.

 * Doc string
```python
"""
  <Purpose>
    Allows the user program to stop execution of the program without
    passing an exit to the main program or calling finally blocks.

  <Arguments>
    None.

  <Exceptions>
    None.

  <Side Effects>
    Interactions with timers and connection / message receiving functions 
    are undefined.   These functions may be called after exit and may 
    have undefined state.

  <Returns>
    None.   The current thread does not resume after exit
"""
```

#### getlock()
----

Returns a lock object that can be used for mutual exclusion and critical section protection.

 * Doc string
```python
"""
  <Purpose>
    Returns a lock object to the user program.    A lock object supports
    two functions: acquire and release.   See threading.Lock() for details

  <Arguments>
    None.

  <Exceptions>
    None.

  <Side Effects>
    None.

  <Returns>
    The lock object.
"""
```

#### lock.acquire(blocking=1)
----

Blocks until the lock is available, then takes it (lock is an object obtained by calling getlock()).

If the optional "blocking" argument is False, the method returns False immediately instead of waiting to acquire the lock; if the lock is available it takes it and returns True, as if it were called with no argument.

 * Doc string
```python
"""
  <Purpose>
    Acquires a lock.

  <Arguments>
    blocking (optional) - if False, returns immediately instead of waiting to acquire the lock.

  <Exceptions>
    None.

  <Side Effects>
    Locks the object.

  <Returns>
    True if the lock was acquired, False otherwise.
"""
```

#### lock.release()
----

Releases the lock. Do not call it if the lock is unlocked.

 * Doc string
```python
"""
  <Purpose>
    Release a lock.

  <Arguments>
    None.

  <Exceptions>
    thread.error if release() is called on an unlocked lock.

  <Side Effects>
    Unlocks the object.

  <Returns>
    None.
"""
```

#### getruntime()
----

Returns a float containing the number of seconds the program has been running.   Note that in very rare circumstances (like the user resetting their clock), this will not produce increasing values.   For the actual time, use NTP (e.g. look in /seattlelib/time.repy for more information in the Seattle SVN trunk).

 * Doc string
```python
"""
  <Purpose>
    Return the amount of time the program has been running.   This is in
    wall clock time.   This function is not guaranteed to always return
    increasing values due to NTP, etc.

  <Arguments>
    None

  <Exceptions>
    None.

  <Side Effects>
    None

  <Returns>
    The elapsed time as float
"""
```

#### randomfloat()
----

Returns a random floating point number between 0.0 (inclusive) and 1.0 (exclusive).   

 * Doc string
```python
"""
  <Purpose>
    Return a random number in the range [0.0, 1.0)

  <Arguments>
    None

  <Exceptions>
    None.

  <Side Effects>
    This function is metered because it may involve using a hardware
    source of randomness.

  <Returns>
    The number (a float)
"""
```

#### settimer(waittime, function, args)
----

Sets a timer that when it expires will start a new thread to call a function with a set of arguments.   The thread is charged to your program when you set the timer (instead of when the timer fires).   This function returns a timer handle that may be used to cancel the timer before the thread is started.   

 * Doc string
```python
"""
  <Purpose>
    Allow the current thread to set an thread to be performed in the future.
    This does not guarantee the thread will be triggered at that time, only
    that it will be triggered after that time.

  <Arguments>
    waittime:
       The minimum amount of time to wait before delivering the thread, in seconds
    function:
       The function to call
    args:
       The arguments to pass to the function.   This should be a tuple or 
       list

  <Exceptions>
    None.

  <Side Effects>
    None.

  <Returns>
    A timer handle, for use with canceltimer
"""
```

#### canceltimer(timerhandle)
----

Try to cancels a timer handle that has not started a thread.   Returns False if the thread has been started already (and so cannot be canceled).   Returns True if the timer was successfully canceled.


 * Doc string
```python
"""
  <Purpose>
    Cancels a timer.

  <Arguments>
    timerhandle:
       The handle of the timer that should be stopped.   Handles are 
       returned by settimer

  <Exceptions>
    None.

  <Side Effects>
    None.

  <Returns>
    If False is returned, the timer already fired or was cancelled 
    previously.   If True is returned, the timer was cancelled
"""
```

#### sleep(seconds)
----

Sleeps the current thread for some time (waits for a specific time before executing any further instructions).   This thread will not consume CPU cycles during this time.    Timing issues that confuse getruntime() may also cause sleep to behave in undefined ways.

 * Doc string
```python
"""
  <Purpose>
    Allow the current thread to pause execution (similar to time.sleep()).
    This function will not return early for any reason

  <Arguments>
    seconds:
       The number of seconds to sleep.   This can be a floating point value

  <Exceptions>
    None.

  <Side Effects>
    None.

  <Returns>
    None.
"""
```
 
#### socket.close()
----
Closes the socket.   Any further local calls to recv / send will result in an exception.

 * Doc string
```python
"""
  <Purpose>
    Closes a socket.   Pending remote recv() calls will return with the
    remaining information.   Local recv / send calls will fail after this.

  <Arguments>
    None

  <Exceptions>
    None

  <Side Effects>
    Pending local recv calls will either return or have an exception.

  <Returns>
    True if this is the first close call to this socket, False otherwise.
"""
```

 
#### socket.recv(bytes)
----
Receives data that was sent by the connected party using send.   Note that this may return less than bytes worth of data.   Also, note that if the other party does ```s.send('hello'); s.send('Guten Tag')```, the other party who calls recv may get 'helloGuten Tag', 'h', or any other subset of the total data.   recv raises an exception when the other side has closed the connection.

 * Doc string
```python
"""
  <Purpose>
    Receives data from a socket.   It may receive fewer bytes than
    requested.

  <Arguments>
    bytes:
      The maximum number of bytes to read.   

  <Exceptions>
    Exception if the socket is closed either locally or remotely.

  <Side Effects>
    This call will block the thread until the other side calls send.

  <Returns>
    The data received from the socket (as a string).   An exception is raised
    when the other side has closed the socket and no more data will arrive.
"""
```
 
#### socket.send(message)
----

Sends data to the connected party.   Note that this may send less than the entire message.   If the connection is disconnected, there is no guarantee that the other party was able to recv the data.  If the other party doesn't call recv, send may block indefinitely.   If the other party closes the connection, send will raise an exception.

 * Doc string
```python
"""
  <Purpose>
    Sends data on a socket.   It may send fewer bytes than requested.

  <Arguments>
    message:
      The string to send.

  <Exceptions>
    Exception if the socket is closed either locally or remotely.

  <Side Effects>
    This call may block the thread until the other side calls recv.

  <Returns>
    The number of bytes sent.   Be sure not to assume this is always the
    complete amount!
"""
```
