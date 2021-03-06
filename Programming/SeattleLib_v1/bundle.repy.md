 = bundle.repy =
----
bundle.repy provides an utility for packaging multiple files into a single file.  This document describes usage via the **Repy API**.   For details on how to use it via the command line, go [wiki:SeattleLib/bundler.py here].

bundle.repy is designed with the intention to allow repy programmers to easily upload static files along with their repy programs.  You can write your program assuming that these files will be present in the local directory, bundle the said files into the program and then distribute it.  The extraction of these files is done for you, before execution.

Bundles are identified by a **.bundle.repy** extension.





## Repy API
----
The bundle is designed to be used directly by a repy program.  It exposes an object-oriented interface to manipulating bundles.  After every method, you may assume that the bundle object is intact and can be used.  


### bundle_bundle.!__init!__()
----
 Purpose
  Creates a new bundle object.  Various bundle operations may then be performed.
 Arguments
  **bundlefn** (str): Where the bundle is, or where to create it.  It must be in the current directory.[[br]]
  **mode** (str): 'r' for readonly, 'w' to create a new bundle, 'a' to modify an existing bundle.[[br]]
  **srcfn** (str/None)(optional): Only used if opening in 'w' mode.  If this is set, overwrite must be False.  The contents of this file will be included with the output bundle.[[br]]
  **overwrite** (bool) (optional): Only used if opening in 'w' mode.  Either this or overwrite may be specified.  If set to True, srcfn must be None.  The contents of the file referred to by bundlefn will be overwritten.  The original contents will be embedded in this new file.
 Side Effects
  If opening in 'w' or 'a', the bundle will be created if it does not already exist in the current directory.
  A file handle will be opened to allow manipulation of the underlying bundle file.
 Exceptions
  If bundle refuses to open, check to make sure that there are no existing open file handles to the bundle's filename.
 Returns
  None


### bundle_bundle.add_file()
----
 Purpose
  Adds the specified file into the bundle.
 Arguments
  **fname** (str): The name of the file to add to the bundle. [[br]]
 Side Effects
  The content of the specified file will be read, encoded using base64, and then written into the end of the bundle.
 Exceptions
  None
 Returns
  None


### bundle_bundle.add_files()
----
 Purpose
  Adds the specified files into the bundle.
 Arguments
  **fnames** (list<str>): The names of the files to add to the bundle. [[br]]
 Side Effects
  The contents of each file that is specified will be read, encoded using base64, and then written into the end of the bundle.
 Exceptions
  None
 Returns
  None


### bundle_bundle.extract_file()
----
 Purpose
  Extracts the specified file from the bundle, and then writes it into the current directory.
 Arguments
  **fname** (str): The name of the file to extract from the bundle. [[br]]
 Side Effects
  The contents of the specified file will be read from the bundle, decoded using base64, and then written into the current directory. [[br]]
  If a file in the current directory has the specified filename, it will be overwritten.
 Exceptions
  None
 Returns
  None


### bundle_bundle.extract_files()
----
 Purpose
  Extracts the specified files from the bundle, and then write them into the current directory.
 Arguments
  **fnames** (list<str>): The names of the files to extract from the bundle. [[br]]
 Side Effects
  The contents of each file that is specified will be read from the bundle, decoded using base64, and then written into the current directory. [[br]]
  Existing files in the current directory that share the same filename as any of the specified files will be overwritten.
 Exceptions
  None
 Returns
  None


### bundle_bundle.extract_string()
----
 Purpose
  Extracts the specified contents of the specified file from the bundle and returns it as a string.
 Arguments
  **fname** (str): The name of the file to extract from the bundle. [[br]]
 Side Effects
  The contents of the specified file will be read from the bundle and then decoded using base64. [[br]]
 Exceptions
  None
 Returns
  The file's contents as a single string.


### bundle_bundle.remove_file()
----
 Purpose
  Removes the specified file from the bundle.
 Arguments
  **fname** (str): The name of the file to remove from the bundle. [[br]]
 Side Effects
  The specified bundle content will be removed from the bundle.  Any data lost is not recoverable. A temporary copy of the bundle is created in the local directory, and is removed when this function returns.
 Exceptions
  None
 Returns
  None


### bundle_bundle.remove_files()
----
 Purpose
  Removes the specified files from the bundle.
 Arguments
  **fnames** (list<str>): The names of the files to remove from the bundle. [[br]]
 Side Effects
  The specified bundle contents will be removed from the bundle.  Any data lost is not recoverable. A temporary copy of the bundle is created in the local directory, and is removed when this function returns.
 Exceptions
  None
 Returns
  None


### bundle_bundle.contents()
----
 Purpose
  Returns the contents of the bundle.
 Arguments
  None
 Side Effects
  None
 Exceptions
  None
 Returns
  A dictionary showing the details of each file contained within the bundle.  An example value is shown below.

Example return value:
```python
{'thefirstfile': {'length': 142}, 'thesecondfile': {'length': 11}}
```



### bundle_clear_bundle_contents()
----
 Purpose
  Clears all traces of a bundle from the specified file.
 Arguments
  **bundlefn** (str): The name of the bundle to clear.  It must be in the current directory. 

  **outputfn** (str/None) (optional): If set, the result will be generated in this file.  Otherwise, overwrite must be set to True. 

  **overwrite** (bool) (optional): If set, the original file will be overwritten with the result.  Otherwise, outputfn must be set. 

 Side Effects
  The specified file will have all of its bundle data removed.  The result will be the file that was used to create the bundle. 

  A temporary file will be created in the current directory, and will be removed when the function returns.
 Exceptions
  None
 Returns
  None




## API Usage Example: Creating an Automated Experiment Script
----
Suppose you plan to use several hundred vessels to parse a two related data sets. You have the test script (**experiment.repy**) written and ready for deployment. You have a script (**generate_experiment.py**) that partitions the data into 1000 pairs (**rawdataAXXX**, **rawdataBXXX**) that the test script will utilize. The vessels will generate three types of output per update, **UsageDataXXX_YYY**, **PerformanceDataXXX_YYY**, and **DatasetQualityXXX_YYY**, where XXX is the experiment #, and YYY is the update #. These vessels will periodically send the results back to the main server, so that not all data is lost if these vessels go down.

For this demonstration, we will work with generating chunk !#143.



### Creating a Bundle
----
In order to create an auto-extracting bundle, we must first instantiate a new Bundle, **experiment143.bundle.repy**. We will add this functionality to our generation script, **generate_experiment.py**.  There are two ways we can go about creating bundles, though only the first method is applicable for the current task:

```repy
# We assume that the experiment doesn't already exist in the current directory.
experiment_bundle = bundle_bundle('experiment143.bundle.repy', 'w', source='experiment.repy')

# Not always the best idea, overwrite the original file.
experiment_bundle = bundle_bundle('experiment143.bundle.repy', 'w', overwrite=True)
```




### Adding Files to a Bundle
----
Now, we add the experiment data to the experiment script. These data elements are **rawDataA143** and **rawDataB143**.  For sake of demonstration, we assume that the data elements will be first written to files in the current directory, named **rawDataA143** and **rawDataB143**. We will bundle these files into **experiment143.bundle.repy**.

```repy
# The filenames of the files to add
dataFnames = ['rawDataA143', 'rawDataB143']
experiment_bundle.add_files(dataFnames)
```




### Adding Strings to a Bundle
----
We will assume that you have a script that will acquire the vessels and start the experiment for you.  **experiment143.bundle.repy** will automatically extract the files that were bundled with it, and start executing. Now suppose that the experiment script is ready to dump its first set of data, **parsed_usage_dataXXX_YYY**, **parsed_performance_dataXXX_YYY**, and **dataset_qualityXXX_YYY**.  These datum are already in strings.  Additionally, we have already instantiated a bundle in the variable **outputbundle**.

```repy
outputbundle.add_string('parsed_usage_data143_000', parsed_usage_data)
outputbundle.add_string('parsed_performance_data143_000', parsed_performance_data)
outputbundle.add_string('dataset_quality143_000', dataset_quality)
```



### Removing Elements from a Bundle
----
We don???t have unlimited disk space on our vessel, so we prefer to overwrite our bundle instead of generating thousands of files. To do this, we will first remove our old bundled data, and then add the new strings into it. We will only demonstrate the removal of the data, as adding them back is done the same way as described above. Once again, the data elements in question are in the bundle with the filenames **parsed_usage_dataXXX_YYY**, **parsed_performance_dataXXX_YYY**, and **dataset_qualityXXX_YYY**.
```repy
old_filenames = ['parsed_usage_data143_000', 'parsed_performance_data143_000', 'dataset_quality143_000']
outputbundle.remove_files(old_filenames)
```



### Checking a Bundle???s Contents
----
Assume that we have a script that will fetch the parsed bundles and place them in the current directory. The collection service will periodically check the current directory for new updates, and extract the parsed data from them. Since we only need to retrieve data, we???ll open this bundle in read-only mode. First we need to check the bundle contents to find which update this is.

```repy
bundle_contents = obtainedbundle.contents()
```

Printing bundle_contents might produce the following output:
```python
{'parsed_usage_data143_000': {'length': 623}, 'parsed_performance_data143_000': {'length': 1342}, 'dataset_quality143_000': {'length': 987231}}
```



### Reading Strings from a Bundle
----
Now, after we extracted the experiment # and update #, we can extract the actual data. The experiment # and update # are stored in **experiment_number** and **update_number**, respectively.
```python
parsed_usage_data = obtainedbundle.extract_string('parsed_usage_data143_000')
parsed_performance_data = obtainedbundle.extract_string('parsed_performance_data143_000')
dataset_quality = obtainedbundle.extract_string('dataset_quality143_000')
```

### Extracting Files from a Bundle
In addition, we can also extract these files into files directly.
We can do them one by one...
```python
fnames_to_extract = ['parsed_usage_data143_000', 'parsed_performance_data143_000', 'dataset_quality143_000']
obtainedbundle.extract_files(fnames_to_extract)
```

Or do them all simultaneously.
```python
obtainedbundle.extract_all_files()
```


### Clearing a Bundle / Original Source File Recovery
Now suppose you forgot to back up all your experiment data, and the only thing you have left is a experiment bundle **experiment143.bundle.repy** that you've previously deployed to a vessel, and you'd like to recover the original experiment script to the file **experiment_copy.repy**.  There are two ways to go about this:

```repy
# Safe recovery, we save the results to an external file
bundle_clear_bundle_contents('experiment143.bundle.repy', 'experiment_copy.repy')

# Not always the best idea, we overwrite the original file
bundle_clear_bundle_contents('experiment143.bundle.repy', overwrite=True)

```

 

## Includes
----
 * [wiki:SeattleLib/base64.repy]
 * [wiki:SeattleLib/serialize.repy]