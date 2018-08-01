
Paralline (Python3)

Full doc at: www.netnotnut.org/paralline   
Licence: MIT  
Contact: paralline@gmx.com  

1. What it does

Reads a directory (or a set of files) and parse all the lines over multiple processes.
Processes receive the lines, run the same script overs those lines
and return their results as an iterable.
Paralline manages these processes and aggregates their results into an iterable
and returns it.

1.1 Finding Help:

$>Python
>>> import paralline
>>> help(paralline)
>>> help(paralline.Paralline)



1.2 Files:

Paralline is designed to run multi Gigabytes long text files.
File lines should have the same schema.
What we call a line is an amount of text data delimited by a separator.
These files can be any semi-structured files like:
log files, json files, .csv, or xml files for instance.
Or any kind of text files that separates contents by the same separator (not need to be \n the default).
Actually Paralline doesnt care about the content as long as it can find the line's sÃ©parator.

1.3 Script:

1/ When run externally as a command

The option: -f (--script_file) is used.
e.g.:
python3 paralline.py /where/is/my/text_file -f /where/is/my/python_file.py

Each Process runs the same script.
This script is a Python file.
This Python file must support the following Function:

def digest(lines):
    <do something>
    return lines # or any iterable or None

This Function may return an iterable or None.
If all calls return None, Paralline would return an empty list.

2/ When run internally as an instance of the Paralline class,

The parameter: fct is used.
e.g.: paralline = Paralline()
result_list = paralline.x(r'/where/is/my/text_file1', r'/where/is/my/text_file2', r'/or/a/directory', ..., fct=list(map(lambda lines:lines)))

"x" is an alias for the function "execute".
Here this lambda just returns lines
Script file could be a Python file like above.

But preferably it is a Python function.
It can be a lambda function or any Python (def) function.
This function must support one argument: lines.
> Because the function is serialized the Python Package: "dill" must be installed:
$ pip install dill


Processes: One can define the number of processes to run,
but by default Paralline would run as much processes as there are cores on the machine.



2. How it does it

No matter what the size of a file is,
Paralline splits the files in packet of lines of same size (shunk-size) as much as possible.
Each packet ends boundary, by a legitimate separator.

Note: a) The default separator is "\n" but it can be provided as a command option "-l" (or "--line_separator").
Or as a the parameter: "line_separator" of the function "x" over the Paralline instance.
b) If Paralline cannot reach the end of the line with the range of "line_max_size" (default 1m),
it tries to go to the end of the file.
But it will raise a Warning:
ParalleleEndLineNotFoundWarning:Contracts:ajustEnd: File: => Unable to seek line separator from end: 1234 within line_max_size: 1M !
You should considere to increase line_max_size.
This is explicite: either you wrong and you provided a bad "line_separator" that doesn't exist in the file,
or line_max_size is not big enough to envelope the longuest line within one file !

A packet may be built of many lines spreaded over one or multiple files.

Actually in Paralline a Packet is a definition that lives within a "contract".
A "contract" defines the disk offset of the first and the last line of the packet.
A "contract" also defines the files where the lines come from.
In a packet they are as many line that are need to reach shunk_size.

Contracts are equally (as much possible) distributed over processes.

- processes iterates over their set of contracts.
- processes read from the files as many lines need for the contract and operate the lines into the regular function.
- The function returns an iterable or None.
- This iterable is streamed up to the Paralline manager which extends it up to the main list.
- The main list is returned by the "x" function.

> Because each process reads the same file system as they access randomly to the set of files,
they must share the same FS.
> This point is a landmark for a future version which plans to support remote worker.


Examples: Into the Paralline directory they are too couple of files:
- test1.txt and test1.py
- test2.txt and test2.py
$> cd <paralline_dir>

a) To run them on command line:
python3 paralline.py test1.txt -f test1.py

b) To run them in Python interpreter:

$> python3
>>> from paralline import Paralline
>>> paralline = Paralline()
>>> paralline.x('test1.txt', fct='test1.py')
or
>>> paralline.x('test1.txt', fct=lambda lines:lines)

e.g.1: If you run this:
python3 paralline.py test1.txt -f test1.py -d


You would see this somthing like that:

Result Lines:
1 Lorem ipsum dolor sit amet, consectetur adipiscing elit,
2 sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
3 Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut
4 aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit
5 esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident,
11 esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident,
12 sunt in culpa qui officia deserunt mollit anim id est laborum
13 Lorem ipsum dolor sit amet, consectetur adipiscing elit,
14 sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
15 Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut
6 sunt in culpa qui officia deserunt mollit anim id est laborum
7 Lorem ipsum dolor sit amet, consectetur adipiscing elit,
8 sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
9 Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut1
10 aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit
16 aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit
17 esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident,
18 sunt in culpa qui officia deserunt mollit anim id est laborum

Lines Count: 19


Please note: It is not supposed to keep the lines order, because processes run concurently on shunks.


Default: Default values are in the file named "paralline.attrs" into the paralline directory.


e.g.2: Your may want to try a big file: python3 paralline.py test2.txt -f test1.py -v10
or: python3 paralline.py test2.txt -f test2.py -p8 -v10


3. How it works (Deeper aspecst: You don't really need to read this)

1/ Launches the QDealer on Host Port.
The QDealer deals queues to the worker processes (that's it).
The Paralline manager and worker processes communicate via Multiprocessing queues.

2/ Redirects Outputs and logs to the Paralline manager.

3/ Tries to Launch as many Workers as required by the "parallele" parameter.

4/ Calculation and Adjustment a) Obtains the total files size:
Reads the size of all files from the set of files or
the provided directories (eventually with the provided prefixes/suffixes).
Sums up all these sizes and obtains : "total_files_size" the total amount of bytes for the whole files.

b) Obtains the total size for one unit of work:
Divides this "total_files_size" per "proc_parallele" and obtains "uow_total_size": the total amount of bytes per process.

c/ Adjusts shunk_size (Safe guard):
if uow_total_size is lesser than shunk_size than uow_total_size/2 becomes shunk_size.
if uow_total_size is greater than shunk_size than shunk_size is kept.

5/ Run Payload on all Workers and Join.
Process receive as much contracts as need to fill up uow_total_size.
"contract" are not bigger than "shunk_size".

