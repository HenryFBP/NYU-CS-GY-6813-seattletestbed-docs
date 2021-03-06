# Building a Security Layer

This assignment will help you understand security mechanisms. You will be guided through the steps of creating a reference monitor using the security layer functionality in Repy V2. A reference monitor is an access control concept that refers to an abstract machine that mediates all access to objects by subjects. This can be used to allow, deny, or change the behavior of any set of calls. One critical aspect of creating a reference monitor is to ensure it cannot be bypassed. While not a perfect solution, it is useful to create test cases to see whether your security layer will work as expected. (The test cases may be turned in as part of the next assignment)

This assignment is intended to prepare you for thinking about security paradigms in a functional way. The ideas of access control and confidentiality have been embedded into the steps of this assignment.






## Overview
----
In this assignment you will create a security layer which stops the user from reading data, writing over existing data, or writing new data to the end of a file without having the permission to do so.  You will write five functions: setread, setwrite, setappend, readat, and writeat.  The setread, setwrite, and setappend functions enable and disable read, write, and append permissions, respectively.  The user may have all, some, or none of the permissions enabled at any given time.  By default, these permissions are disabled.  The readat function will allow the user to read data from the file if and only if the user has read permissions.  The writeat function will allow the user to overwrite existing data if and only if write permissions are enabled.  Similarly, the user may append new data to the file if and only if append permissions are enabled.  Attempting to overwrite or append data when the user does not have permissions to do so will result in nothing being written.  If both append and write permissions are disabled when a writeat is called, a ValueError is raised.  You will do this by adding security rules to the functions available for setting the permissions, as well as reading from and writing to a file.Think of this as a method to write some data securely into a file. The future of the system depends on your ability to write secure code!

Three design paradigms are at work in this assignment: accuracy, efficiency, and security.

 * Accuracy: The security layer should only stop certain actions from being blocked. All other actions should be allowed. For example, If a user tries to overwrite data and append new data in the same write, and has permissions to write, but not append, then the corresponding data would be overwritten, but would stop at the end of the file and would not append anything.

 * Efficiency: The security layer should use a minimum number of resources, so performance is not compromised. This means you should not do things like re-read the file before each write.

 * Security: The attacker should not be able to circumvent the security layer. Hence, if the data can be written without permission, the security is compromised.



### Getting Python
----
Please note you must have Python 2.5, 2.6, or 2.7 to complete this assignment. Instructions on how to get Python for Windows can be found [InstallPythonOnWindows here].  If you are using Linux or a Mac it is likely you already have Python. In order to verify this, simply open a terminal and type ```python```.  Please check its version on the initial prompt.

**Note:**If you are using Windows, you will need python in your path variables.  A tutorial on adding variables to your path in Windows can be found at [http://www.computerhope.com/issues/ch000549.htm]

### Getting RepyV2
----
The preferred way to get Repy is ''installing from source''. For this, you check out the required Git repositories, and run a build script. You can always update the repos later, and rebuild, so that you get the latest stable version of the Repy runtime.

Here's how to do that. Assuming you are running on a Unixoid OS,

```
# Create a directory for the required Git repositories
mkdir SeattleTestbed
cd SeattleTestbed

# Check out the repos required for building Repy
git clone https://github.com/SeattleTestbed/repy_v2.git

# Prepare a build directory, and build into it
cd repy_v2/scripts
mkdir ~/path/to/build/dir
python initialize.py
python build.py ~/path/to/build/dir
```

Once the build script finished, `~/path/to/build/dir` contains a ready-to-use copy of the RepyV2 runtime!
----
(If you cannot install from source, [attachment:repyv2_commit_3499642.zip here] is a tarball including a pre-built runtime.)
----



Use the command found below in order to run Repy files:

```python repy.py restrictions.default encasementlib.r2py [security_layer].r2py [program].r2py``` 

Please note Repy files end in extension `.r2py`.   

In order to test whether or not these steps worked, please copy and paste the code found below for the sample security layer and sample attack.  Where you can find the sample security layer and sample attack:

 * Sample security layer: [wiki:EducationalAssignments/PermissionsPartOne#Abasicandinadequatedefense BasicAndInadequateDefense] 
 * Sample attack layer:  [wiki:EducationalAssignments/PermissionsPartOne#Testingyoursecuritylayer Testingyoursecuritylayer]

If you got an error, please go through the trouble shooting section found below.

### Troubleshooting Repy code
If you can't get Repy files to run, some of the following common errors may have occurred:

 * using `print` instead of `log`:

Repy is a subset of Python, but its syntax is slightly different.  For example, Python's `print` statement cannot be used; Repy has `log` for that. For a full list of acceptable syntax please see wiki:RepyV2API.

 * command line errors:

**files are missing:** In the above command line call, you must have `repy.py`, restrictions.default encasementlib.r2py, the security layer and the program you want to run in the current working directory.  If any or all of the above files are not in that directory then you will not be able to repy files.  If this is the case, it is likely you are trying to run repy files from the wrong directory.  Since `repy.py` exists in multiple directories, it is possible to run `.r2py` files from directories other than the the one you are supposed to use.  

<!--
AR: This doesn't apply when building from source or getting the runtime tarball only (it does for clearinghouse downloads).
 * Downloading the wrong version of seattle:

Seattle is operating system dependent.  If you download the Windows version, you need to use the Windows command line.  For Windows 7 this is PowerShell.  You can open a new terminal by going to start, search, type powershell.  If you downloaded the Linux version you must use a Linux OS and Linux terminal.  

AR: This is obviously outdated.
Advanced trouble shooting:

To run the unit test, which will automatically tell you if you have errors with your installation please see:

 * [wiki:RepyV2CheckoutAndUnitTests]
-->


### Tutorials for Repy and Python
----
Now that you have Repy and Python, you may need a refresher on how to use them.  The following tutorials are excellent sources of information.

 * Python tutorial: **[http://docs.python.org/tutorial/]**
 * Seattle tutorial: **[https://seattle.poly.edu/wiki/PythonVsRepy]**
 * list of RepyV2 syntax: **[wiki:RepyV2API]**



## Building the security layer
----
**[wiki:RepyV2SecurityLayers]** explains the syntax used to build a reference monitor.  The general tutorials above will aid in looking up other details about Repy.  Remember, you have no idea how the attacker will try to penetrate your security layer, so it is important that you leave nothing to chance!  Your reference monitor should try to stop every attack you can think of.  Not just one or two.  A good reference monitor will do this and incorporate the design paradigms of accuracy, efficiency and security defined above.


### A basic (and inadequate) defense

Time to start coding!  Let's inspect a basic security layer.  

```
"""
This security layer interposes on a textfile and gives it read, write, or append access.
A user can only read, write, or append to that file when that specific mode is enabled for that file.   Multiple files can be be simultaneously opened and have different modes.

Read-only: You may read the contents of a file, but not write or append.
If you attempt to read when read mode is disabled, a ValueError is raised.

Write-only: You may overwrite the contents of a file, but not append to the file.
If you attempt to do so, the file should be overwritten to the length of the file, and then writing stops.

Append-only: You may append data to the end of the file, but not overwrite 
anything that is already written.   If a write starts before the end of the
file, but continues afterward, only the data after the current end of
file should be written.

More than one mode may be enabled at once, e.g. Write and Append allows you to overwrite existing file data and append data to the end of the file.

writeat must return a ValueError if and only if both write and append are 
blocked.  readat must return a value error if and only if read is blocked.   

For efficiency purposes, you *MUST NOT* call readat inside of writeat.   It
is fine to call readat during file open if desired.


Extra credit (?): remember permissions for files after they are closed and
reopened.   This would include situations where your security layer is itself
closed and reopened (for example, if the system restarts).   You can assume
that all reads must go through your security layer and so you can add an 
additional file or modify the file format.

Note:
    This security layer uses encasementlib.r2py, restrictions.default, repy.py and Python
    Also you need to give it an application to run.
    This security layer never runs explicitly but instead interposes functions
    from above layers.
    
    """ 


TYPE="type"
ARGS="args"
RETURN="return"
EXCP="exceptions"
TARGET="target"
FUNC="func"
OBJC="objc"

class SecureFile():
  def __init__(self,file):
    # globals
    mycontext['debug'] = False   
    mycontext['read'] = False
    mycontext['write'] = False
    mycontext['append'] = False
    # local (per object) reference to the underlying file
    self.file = file

  def setread(self,enabled):
    mycontext['read'] = enabled

  def setwrite(self,enabled):
    mycontext['write'] = enabled

  def setappend(self,enabled):
    mycontext['append'] = enabled

  def readat(self,bytes,offset):
    if not mycontext['read']:
      raise ValueError
    return self.file.readat(bytes,offset)

  def writeat(self,data,offset):
    if not mycontext['write']:
      return
    if not mycontext['append']:
      return
    self.file.writeat(data,offset)


  def close(self):
    return self.file.close()

sec_file_def = {"obj-type":SecureFile,
                "name":"SecureFile",
                "setread":{TYPE:FUNC,ARGS:[bool],EXCP:Exception,RETURN:(type(None)),TARGET:SecureFile.setread},
                "setwrite":{TYPE:FUNC,ARGS:[bool],EXCP:Exception,RETURN:(type(None)),TARGET:SecureFile.setwrite},
                "setappend":{TYPE:FUNC,ARGS:[bool],EXCP:Exception,RETURN:(type(None)),TARGET:SecureFile.setappend},
                "readat":{TYPE:FUNC,ARGS:((int,long,type(None)),(int,long)),EXCP:Exception,RETURN:str,TARGET:SecureFile.readat},
                "writeat":{TYPE:FUNC,ARGS:(str,(int,long)),EXCP:Exception,RETURN:(int,type(None)),TARGET:SecureFile.writeat},
                "close":{TYPE:FUNC,ARGS:None,EXCP:None,RETURN:(bool,type(None)),TARGET:SecureFile.close}
           }

def secure_openfile(filename, create):
  f = openfile(filename,create)
  return SecureFile(f)

CHILD_CONTEXT_DEF["openfile"] = {TYPE:OBJC,ARGS:(str,bool),EXCP:Exception,RETURN:sec_file_def,TARGET:secure_openfile}

secure_dispatch_module()

```

### Using the example layer

Keep in mind the above security layer would not prevent writing without permissions, and only protects reading without permissions from certain attacks.  There are a bunch of tricks that can be used in order to circumvent this security layer easily. For instance, the above reference monitor isn't thread safe. For an introduction to thread safety please read [wiki/Thread_safety](http://en.wikipedia.org/wiki/Thread_safety).



### Code analysis

The above functions attempt to follow the design principles of accuracy, efficiency, and security. However they do so inadequately.For example the data will not be written at all unless both permissions to write and append are enabled. It is more important to make a security system that is infeasible to break, rather than impossible to break. However this security layer does not yet create the necessary infeasibility requirements for most attacks. Thus we see that there is a grey area of what is an acceptable level of impedance.



### Testing your security layer
----
In this part of the assignment you will pretend to be an attacker. Remember the attacker's objective is to bypass permissions. By understanding how the attacker thinks, you will be able to write better security layers. Perhaps while attacking your security layer you will think of a new mitigation that should have been implemented. Keep in mind attacks are attempts to mitigate a given security protocol. If even one case succeeds, then your security layer has been compromised. Thus the attack you write should include several methods of attempting to bypass permissions. An example of an attack is found below:

```
if "look.txt" in listfiles():
  removefile("look.txt")
myfile=openfile("look.txt",True)  #Open a file

#Write some data to file
myfile.setread(True)
myfile.setwrite(True)
myfile.setappend(True)
myfile.writeat("This is secure",0)

# This read is in the region written by writeat and should not be blocked...
x=myfile.readat(14,0)
log(x)

#Try to overwrite data without permissions
myfile.setwrite(False)
myfile.writeat("SECURITY",8)   # no exception because append is allowed...   (seems odd)
y=myfile.readat(16,0)
if y == "This is secureTY":
  #If security layer successful, this should pass
  pass
else:
  #If security layer fails
  log("Data compromised!")

#Close the file
myfile.close()
```

**Note:** All attacks should be written as Repy files, using the .r2py extension.

#### Code Analysis
It is important to keep in mind that only lowercase file names are allowed. So in the above code, specifically:

```
# Open a file
myfile=openfile("look.txt",True)
```

look.txt is a valid file name, however Look.txt is not. Examples of other invalid files names are, look@.txt, look/.txt, and look().txt. Essentially all non-alphanumeric characters are not allowed.

This code attempts to read the ???This is secure??? from the file directly. First the file is opened using myfile=openfile("look.txt",True). Next myfile.writeat writes some data to the file with permissions. Then the write permissions are removed, and writeat is once again used to write some data to the file, some of which attempts to overwrite the existing data. If the security layer is written properly, only the appended data will be written.


### Running your security layer
----
Finally, type the following commands at the terminal to run your security layer with your attack program

```python repy.py restrictions.default encasementlib.r2py [security_layer].r2py [attack_program].r2py ```

Make sure you went through the "How to get RepyV2" section!


# Notes and Resources
----
   
 * A list of command line utilities for windows can be found at **[http://commandwindows.com/command3.htm]**

 * A list of command line utilities for linux/apple/powershell **[http://www.pixelbeat.org/cmdline.html]**

 * A tutorial on how to write security layers can be found [wiki:RepyV2SecurityLayers here].  At the end of the tutorial there is a second example on **how to test security layers**. 

 * For a complete list of syntax in Repyv2 please visit: **[wiki:RepyV2API]**
 
 * The following link is an excellent source for information about security layers: **[https://ssl.engineering.nyu.edu/papers/cappos_seattle_ccs_10.pdf]**

 * **[repy_v2/benchmarking-support/allnoopsec.py](https://seattle.poly.edu/browser/seattle/branches/repy_v2/benchmarking-support/allnoopsec.py)** is an empty security layer that doesn't perform any operations.

 * **[repy_v2/benchmarking-support/all-logsec.py](https://seattle.poly.edu/browser/seattle/branches/repy_v2/benchmarking-support/all-logsec.py)** is security layer that works for logging functions.

 * **Note:** It is possible to add multiple security layers to Repy, this may be useful for testing different mitigations separately.  This is done with the following command at the terminal:

```python repy.py restrictions.default encasementlib.r2py [security_layer1].r2py [security_layer2].r2py [security_layer3].r2py [program].r2py```

**Your security layer should produce no output!! **If it encounters an attempt to read the secure data, you should raise a ValueError exception and allow execution to continue.

 * In repy log replaces print from python.  This may be helpful when testing if Repy installed correctly.


# Extra Credit
----
Remember permissions for files after they are closed and reopened.   This would include situations where your security layer is itself closed and reopened (for example, if the system restarts).   You can assume that all reads must go through your security layer and so you can add an additional file or modify the file format.



# What to turn in?
----

 * Turn in a repy file called reference_monitor_[ polyusername ].r2py with all letters in lowercase. Be sure your reference monitor never produces output. It should raise a ValueError if the calls should be blocked, but never, ever call log() to output information.   Never raise unexpected errors.

 * For extra credit turn in a second repy file called extra_credit_[polyusername].r2py
