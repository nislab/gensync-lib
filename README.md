# GenSync-lib

GenSync-lib provides a library for synchronizing data _efficiently_ across different machines on MacOS and Linux.  The library
includes implmenetations of several state-of-the-art protocols that optimize different metrics, including:
* ___Communication___ - total number of bytes transmitted
* ___Rounds___ - the amount of back-and-forth needed to complete synchronization
* ___Computation___ - computing resources required
* ___Memory___ - amount of physical memory needed

The current version is 2.0.5

<p align="center">
<img src="images/sync.jpg" width=300 center>
</p>
------------------------------

## Table Of Contents
- [Installation/Compilation](#Installation/Compilation)
- [Usage](#UseInstructions)
    - [Examples](#Examples)
    - [Builder Parameters](#BuilderParameters)
    - [Sync Types](#SyncTypes)
- [References](#References)
    - [Contributors](#Contributors)

------------------------------
<a name="Installation/Compilation"></a>
### Installation/Compilation:
The library is supported on linux/unix or mac (not Windows) and sports several dependencies, which should be installed first:
* [NTL](http://www.shoup.net/ntl/) - A library for doing Number Theory (>9.5)
  - ptheads - may be required depending on how NTL is configured
  - [gmp](https://gmplib.org/) - may be required depending on how NTL is configured
* [cppunit](http://cppunit.sourceforge.net/doc/cvs/index.html) - For testing the code
* [cmake](https://cmake.org) - For building the library

There are several ways to install the library:
#### MacPorts
The library is available on [MacPorts](https://www.macports.org/) for Apple Mac computers and can be installed with:

```
$ port install gensyc
```

This should also recursively install dependencies.

#### Copmilation from sources
The library can be compiled from scratch on a linux/unix or Mac computer as follows:
```
$ git clone --recurse-submodules git@github.com:nislab/gensync-lib.git
$ cd gensync-lib
$ cmake .
```

To install the library and include headers in their default locations (with root access), you can also run:
```
$ make install
```

------------------------------
<a name="UseInstructions"></a>
## Usage

<a name="Examples"></a>
### Examples

Two files are provided for demonstrating the library in use:
   * *TryMe* - reconciles two hosts on two separate processes.
   * *TryMe2* - allows the user to select various synchronization parameters from the command-line.
   * *main* - provides a command-line interface to much of the library's funcionality

Compilation depends on the location of GenSync include directory `$GEN_INCLUDE` and library directory `$GEN_LIB`,
which depend on how the library was installed:

- *MacPorts:*  The default installation directory is typically `/opt/local`.
- *cmake:*  From the folder in which you cloned the library source, you can find:
   - the include directory is `gensync-lib/gensync-core/include`
   - the library directory is `gensync-lib`
- *make install:* If you installed the library (with root privileges), then:
   - the include directory is typically `/usr/local/include`.
   - the library directory is typically `/usr/local/lib`.

#### TryMe
This program launches two processes, connected by a network socket:
* The first process (host 1) contains a set with elements 'a', 'b', and 'c'.
* The second process (host 2) contains a set of elements 'b' and 'd'.

##### Compilation
The program can be compiled from the root `gensync-lib` directory with
```shell
$ g++ -I${GEN_INCLUDE} -L${GEN_LIB} -std=c++17 gensync-core/src/TryMe.cpp -lgensync -lntl -o tryme
$ ./tryme
```

##### Output
The output from the program shows both hosts with the same sets (note that the order of elements within a set does not matter):
```
host 1 now has a b c d 
host 2 now has b d c a 
```

#### TryMe2
A more complicated example allows the user to select various synchronization parameters from the command-line.

The program can be compiled with
```shell
$ g++ -I${}GEN_INCLUDE} -L${GEN_LIB} -std=c++17 gensync-lib/gensnc-core/src/TryMe2.cpp -lgensync -lntl -o tryme2
```

To run, open two terminals.  In one issue the command:

```shell
$ ./tryme2 server CPISync
connecting on port 8001...
sync succeeded.
```

In a second, issue the command:

```shell
$ ./tryme2 client CPISync
listening on port 8001...
sync succeeded.
```

#### main
This provides a command-line interface to many of the library's features:
```shell
$ g++ -I${}GEN_INCLUDE} -L${GEN_LIB} -std=c++17 gensync-lib/gensnc-core/src/main.cpp -lgensync -lntl -o main
```
##### Output
```shell
$ ./main --help
Usage:  ./main   [(-f | --file) <filename>]
   [(-s | --string) <string>]
   [(-h | --host) <hostname>]
   [(-o | --port] <port number>]
   [(-e | --perr) <max error prob> 
   [(-m | --mbar) <int>
   [(-b | --bits) <int>
   [(-p | --partition) <int>
   [(-n | --numElems) <int>
   [--cpi | --oneway | --prob | --inter | --iblt]
   [--nohash]
   [--integer]
   [--client | --server]
   [--help]
```

### Progammatic use:
To write code that utilizes the library:

0. Include the relevant headers:
    ```cpp
    #include <GenSync/(Aux/Data/Communicants/Syncs)/(The module you want to use)>
    ```

1. Initialize a `GenSync` object with the constructor or builder helper class on the client and server machine

    ```cpp
       GenSync::Builder builder = GenSync::Builder().
          setProtocol(GenSync::SyncProtocol::GenSync).  //GenSync,InterCPISync, ProbCPISync, IBLTSync, FullSync, etc.
          setComm(GenSync::SyncComm::socket). //CommSocket or CommString
          setPort(8001).
          setHost(localhost).
       
          //Set the parameters for your specific sync type (Not the same for every protocol. See GenSync build parameters section)
          setBits(64). // Bits per element
          setMbar(256). // Max differences to sync
          setErr(7); // -log_2(prob of error) to allow for errors that might otherwise underflow
          
       GenSync mySyncClientOrServer = builder.build();
    ```

2. Add elements to your GenSyncs (If you need to add large elements use the ZZ class from NTL)
    * You may manually create a shared_ptr<DataObject> (Data/DataObject.h) or pass a data type compatible with DataObject and one will be automatically created for you, returning a pointer to the newly created DataObject
   ```cpp
       mySyncClientOrServer.addElem(myZZ);
       mySyncClientOrServer.addElem(myLong);
       mySyncClientOrServer.addElem(myInt);
       mySyncClientOrServer.addElem(myMultiSet); // Multisets are serialized for use in IBLTSetOfSets. This does NOT add each element in the multiset to your sync.
   ```


3. Run `serverSyncBegin` and `clientSyncBegin` on the server and client respectively
   ```cpp
       mySyncServer.serverSyncBegin(0); //Add the index of the sync you would like to perform
       mySyncClient.clientSyncBegin(0);  //Multiple syncs or communicants may be added to one GenSync
   ```

4. Collect relevant statistics
   ```cpp
      //The sync index is decided by the order you added your sync in
      //If your GenSync has only 1 sync the index is 0
       mySyncClient.printStats(syncIndex); //Returns a string of formatted stats about your sync
       mySyncClient.mySyncVec[syncIndex]->getName(); //Retruns the name and parameters for your sync
       mySyncClient.getXmitBytes(syncIndex); //Returns the number of bytes transmitted by this sync
       mySyncClient.getRecvBytes(syncIndex); //Returns the number of bytes received by this sync
       mySyncClient.getCommTime(syncIndex); //Returns the amount of time in seconds that the sync spent sending and receiving info through a socket
       mySyncClient.getIdleTime(syncIndex); //The amount of time spent waiting for a connection or for a peer to finish computation
       mySyncClient.getCompTime(syncIndex); //The amount of time spent doing computations

       
   ```

<a name="BuilderParameters"></a>
#### GenSync Builder Parameters:
* **setProtocol:** Set the protocol that your sync will execute (from the list above)
    * *All syncs*
*  **setComm:** Set the communication method your sync will use (CommSocket and CommString). Comm String is for local testing
    * *All Syncs*
*  **setPort & setHost:** Set the port & host that your socket will use
    * *Any socket based syncs*
*  **setIOString:** Set the string with which to synchronize
    * *Only for CommString based syncs*
*  **setBits:** The number of bits that represent each element in the set
    * *All syncs except FullSync*
*  **setMbar:** The maximum number of symmetric differences that can be synced by a GenSync
    * *All GenSync variants*
*  **setErr:** The negative log base 2 of the probability of error you would like to use to bound your sync
    * *All GenSync variants*
* **setHashes:** If true, elements are hashed non-trivially (Must be true to synchronize multisets)
    * *All GenSync variants*
* **setNumPartitions:** The number of partitions that InterCPISync should recurse into if it fails
    * *InteractiveCPISync*
* **setExpNumElems:** The maximum number of differences that you expect to be placed into your IBLT. If you are doing IBLTSetOfSets this is the number of child sets you expect
    * *IBLTSync, OneWayIBLTSync & IBLTSetOfSets*
* **setExpNumElemChild:** Set the upper bound for number of elements in each child set
    * *IBLTSetOfSets*
* **setDataFile:** Set the data file containing the data you would like to populate your GenSync with
    * *Any sync you'd like to do this with*

<a name="SyncTypes"></a>
#### Sync Types:
* **Included Sync Protocols (Sets and Multisets):**
    * CPISync
        * Sync using the protocol described [here](http://ipsit.bu.edu/documents/ieee-it3-web.pdf). The maximum number of differences that can be reconciled must be bounded by setting mBar. The server does the necessary computations while the client waits, and returns the required values to the client
    * CPISync_OneLessRound
        * Perform CPISync with set elements represented in full in order to reduce the amount of rounds of communication by one (No hash inverse round of communication). The server does the necessary computations while the client waits, and returns the required values to the client
    * OneWayCPISync
        * Perform CPISync in one direction, only adding new elements from the client to the server. The client's elements are not updated. The server does the necessary computations and determines what elements they need to add to their set. The client does not receive a return message and does not have their elements updated
    * ProbCPISync
        * Perform CPISync with a given mBar but if the amount of differences is larger than that, double mBar until the sync is successful. The server does the necessary computations while the client waits, and returns the required values to the client
    * InteractiveCPISync
        * Perform CPISync but if there are more than mBar differences, divide the set into `numPartitions` subsets and attempt to CPISync again. This recurses until the sync is successful. The server does the necessary computations while the client waits, and returns the required values to the client
    * FullSync
        * The client sends the server its set contents and the server determines what elements it needs from the clients set. The server also determines what elements the client needs and sends them back.
    * IBLTSync
        * Each peer encodes their set into an [Invertible Bloom Lookup Table](https://arxiv.org/pdf/1101.2245.pdf) with a size determined by NumExpElements and the client sends their IBLT to their per. The differences are determined by "subtracting" the IBLT's from each other and attempting to peel the resulting IBLT. The server peer then returns the elements that the client peer needs to update their set
    * OneWayIBLTSync
        * The client sends their IBLT to their server peer and the server determines what elements they need to add to their set. The client does not receive a return message and does not update their set
    * CuckooSync
        * Each peer encodes their set into a [cuckoo filter](https://www.cs.cmu.edu/~dga/papers/cuckoo-conext2014.pdf). Peers exchange their cuckoo filters. Each host infers the elements that are not in its peer by looking them up in the peer's cuckoo filter. Any elements that are not found in the peer's cuckoo filter are sent to it.
* **Included Sync Protocols (Set of Sets):**
   * IBLT Set of Sets
       * Sync using the protocol described [here](https://dl.acm.org/doi/abs/10.1145/3196959.3196988). This sync serializes an IBLT containing a child set into a bitstring where it is then treated as an element of a larger IBLT. Each host recovers the IBLT containing the serialized IBLTs and deserializes each one. A matching procedure is then used to determine which child sets should sync with each other and which elements they need. If this sync is two way this info is then sent back to the peer node. The number of differences in each child IBLT may not be larger than the total number of sets being synced

<a name="References"></a>
## Reference:
If you use this software in your works, please cite the following paper ([DOI](http://doi.org/10.1109/TNSM.2022.3164369)):

`Boškov, Novak, Ari Trachtenberg, and David Starobinski. "Gensync: A new framework for benchmarking and optimizing reconciliation of data." IEEE Transactions on Network and Service Management 19.4 (2022): 4408-4423.`

or in [BibTex](https://scholar.googleusercontent.com/scholar.bib?q=info:pDq08RhPXS4J:scholar.google.com/&output=citation&scisdr=ChXrdsEpEOrg4DdvVFI:ABFrs3wAAAAAZKhpTFLD0mjTsQegzm0Ycic1Fjc&scisig=ABFrs3wAAAAAZKhpTPmcD74OAT0yhaxuOkj4mDY&scisf=4&ct=citation&cd=-1&hl=en):
``` bibtex
@article{bovskov2022gensync,
  title={Gensync: A new framework for benchmarking and optimizing reconciliation of data},
  author={Bo{\v{s}}kov, Novak and Trachtenberg, Ari and Starobinski, David},
  journal={IEEE Transactions on Network and Service Management},
  volume={19},
  number={4},
  pages={4408--4423},
  year={2022},
  publisher={IEEE}
}
```
### Additional literature
The code, explanation, references, API, and a demo can be found on this
web page.  If you use this work, please cite any relevant papers below.

#### The main theoretical bases for the approaches in this work are:
* Y. Minsky, A. Trachtenberg, and R. Zippel,
  "Set Reconciliation with Nearly Optimal Communication Complexity",
  IEEE Transactions on Information Theory, 49:9.
  <http://ipsit.bu.edu/documents/ieee-it3-web.pdf>

* Y. Minsky and A. Trachtenberg,
  "Scalable set reconciliation"
  40th Annual Allerton Conference on Communication, Control, and Computing, 2002.
  <http://ipsit.bu.edu/documents/BUTR2002-01.pdf>

#### Relevant applications and extensions can be found at:
* D. Starobinski, A. Trachtenberg and S. Agarwal,
  "Efficient PDA synchronization"
  IEEE Transactions on Mobile Computing 2:1, pp. 40-51 (2003).
  <http://ipsit.bu.edu/documents/efficient_pda_web.pdf>

* S. Agarwal, V. Chauhan and A. Trachtenberg,
  "Bandwidth efficient string reconciliation using puzzles"
  IEEE Transactions on Parallel and Distributed Systems 17:11,pp. 1217-1225 (2006).
  <http://ipsit.bu.edu/documents/puzzles_journal.pdf>

*  M.G. Karpovsky, L.B. Levitin. and A. Trachtenberg,
   "Data verification and reconciliation with generalized error-control codes"
   IEEE Transactions on Information Theory 49:7, pp. 1788-1793 (2003).

* More at <http://people.bu.edu/trachten>.

#### Additional algorithms:
* Eppstein, David, et al. "What's the difference?: efficient set reconciliation without
  prior context." ACM SIGCOMM Computer Communication Review 41.4 (2011): 218-229.

* Goodrich, Michael T., and Michael Mitzenmacher. "Invertible bloom lookup tables."
  49th Annual Allerton Conference on Communication, Control, and Computing (Allerton), 2011.

* Mitzenmacher, Michael, and Tom Morgan. "Reconciling graphs and sets of sets."
  Proceedings of the 37th ACM SIGMOD-SIGACT-SIGAI Symposium on Principles of Database
  Systems. ACM, 2018.

<a name="Contributors"></a>
### Contributors:

Elements of the GenSync project code have been worked on, at various points, by:

* Ari Trachtenberg
* Sachin Agarwal
* Paul Varghese
* Jiaxi Jin
* Jie Meng
* Alexander Smirnov
* Eliezer Pearl
* Sean Brandenburg
* Zifan Wang
* Novak Boškov
* Xingyu Chen
* Nathan Strahs



# Acknowledgments:
* NSF
* Professors:
    * Ari Trachtenberg, trachten@bu.edu, Boston University
    * David Starobinski, staro@bu.edu, Boston University
