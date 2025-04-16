ppft
====
distributed and parallel Python

About Ppft
----------
``ppft`` is a friendly fork of Parallel Python (``pp``). ``ppft`` extends Parallel Python to provide packaging and distribution with ``pip`` and ``setuptools``, support for Python 3, and enhanced serialization using ``dill.source``. ``ppft`` uses Parallel Python to provide mechanisms for the parallel execution of Python code on SMP (systems with multiple processors or cores) and clusters (computers connected via network).

Software written in Python finds applications in a broad range of the categories including business logic, data analysis, and scientific calculations. This together with wide availability of SMP computers (multi-processor or multi-core) and clusters (computers connected via network) on the market create the demand in parallel execution of Python code.

The most common way to write parallel applications for SMP computers is to use threads. However, the Python interpreter uses the GIL (Global Interpreter Lock) for internal bookkeeping, where the GIL only allows one Python byte-code instruction to execute at a time, even on an SMP computer. Parallel Python overcomes this limitation, and provides a simple way to write parallel Python applications. Internally, processes and IPC (Inter Process Communications) are used to organize parallel computations. Parallel Python is written so that the details and complexity of IPC are handled internally, and the calling application just submits jobs and retrieves the results. Software written with Parallel Python works in parallel on many computers connected via a local network or the Internet. Cross-platform portability and dynamic load-balancing allows Parallel Python to parallelize computations efficiently even on heterogeneous and multi-platform clusters. Visit http://www.parallelpython.com for further information on Parallel Python.

``ppft`` is part of ``pathos``, a Python framework for heterogeneous computing.
``ppft`` is in active development, so any user feedback, bug reports, comments,
or suggestions are highly appreciated.  A list of issues is located at https://github.com/uqfoundation/ppft/issues, with a legacy list maintained at https://uqfoundation.github.io/project/pathos/query.


Major Features
--------------
``ppft`` provides:

* parallel execution of Python code on SMP and clusters
* easy-to-understand job-based parallelization
* automatic detection of the number of effective processors
* dynamic processor allocation (at runtime)
* low overhead for jobs with the same function (through transparent caching)
* dynamic load balancing (jobs are distributed at runtime)
* fault-tolerance (if a node fails, tasks are rescheduled on the others)
* auto-discovery of computational resources
* dynamic allocation of computational resources
* SHA based authentication for network connections
* enhanced serialization, using ``dill.source``


Current Release
[![Downloads](https://static.pepy.tech/personalized-badge/ppft?period=total&units=international_system&left_color=grey&right_color=blue&left_text=pypi%20downloads)](https://pepy.tech/project/ppft)
[![Conda Downloads](https://img.shields.io/conda/dn/conda-forge/ppft?color=blue&label=conda%20downloads)](https://anaconda.org/conda-forge/ppft)
[![Stack Overflow](https://img.shields.io/badge/stackoverflow-get%20help-black.svg)](https://stackoverflow.com/questions/tagged/parallel-python)
---------------
The latest released version of ``ppft`` is available from:
    https://pypi.org/project/ppft

``ppft`` is distributed under a 3-clause BSD license, and is a fork of ``pp-1.6.6``.


Development Version
[![Support](https://img.shields.io/badge/support-the%20UQ%20Foundation-purple.svg?style=flat&colorA=grey&colorB=purple)](http://www.uqfoundation.org/pages/donate.html)
[![Documentation Status](https://readthedocs.org/projects/ppft/badge/?version=latest)](https://ppft.readthedocs.io/en/latest/?badge=latest)
[![Build Status](https://app.travis-ci.com/uqfoundation/ppft.svg?label=build&logo=travis&branch=master)](https://app.travis-ci.com/github/uqfoundation/ppft)
[![codecov](https://codecov.io/gh/uqfoundation/ppft/branch/master/graph/badge.svg)](https://codecov.io/gh/uqfoundation/ppft)
-------------------
You can get the latest development version with all the shiny new features at:
    https://github.com/uqfoundation

If you have a new contribution, please submit a pull request.


Installation
------------
``ppft`` can be installed with ``pip``::

    $ pip install ppft

To include enhanced serialization, using ``dill.source``, install::

    $ pip install ppft[dill]

If Parallel Python is already installed, it should be uninstalled before ``ppft`` is installed -- otherwise, ``import pp`` may point to the original and not to the ``ppft`` fork.


Requirements
------------
``ppft`` requires:

* ``python`` (or ``pypy``), **>=3.8**
* ``setuptools``, **>=42**

Optional requirements:

* ``dill``, **>=0.4.0**


Basic Usage
-----------
``ppft`` is a fork of the Parallel Python package (``pp``) that has been
converted from Python 2 to Python 3, made PEP 517 compliant, and augmented
with ``dill.source``.  For simple parallel execution, first create a job
``Server`` where the number nodes available is autodetected::

    >>> import ppft as pp
    >>> job_server = pp.Server()

The number of nodes can be specified by passing an int as the first argument
when creating the server (i.e. ``Server(4)`` creates a server with four nodes).
The server uses ``submit`` to execute jobs in parallel. ``submit`` takes a
function, a tuple of the arguments to pass to the function, a tuple of any
functions used but not imported in the function, and a tuple of any modules
required to produce the function:: 

    >>> import math
    >>> f1 = job_server.submit(math.sin, (math.pi/2,), (), ('math',))
    >>> f2 = job_server.submit(min, (3.2, 10.0, 7.5), (), ())
    >>> f3 = job_server.submit(sum, ([1,2,3],), (), ())

The functions are serialized by ``dill.source`` (as opposed to ``dill``), by
extracting and passing the source code to the server. The server compiles
and executes the source code, and then calls the function with the arguments
passed in the tuple. Any function and module dependencies are imported
before ``exec`` is called on the source code. Results are retrieved by
calling the object returned from ``submit``::

    >>> f1()
    1.0
    >>> f2()
    3.2
    >>> f3()
    6

Job server execution statistics can be printed with::

    >>> job_server.print_stats()
    Job execution statistics:
     job count | % of all jobs | job time sum | time per job | job server
             3 |        100.00 |       0.0051 |     0.001684 | local
    Time elapsed since server creation 148.48280715942383
    0 active tasks, 4 cores

``ppft`` also can execute jobs on remote computational nodes, if a ``ppserver``
is first started on the node. Here the ``ppserver`` is started on 127.0.0.1,
and will listen on port 35000::

    $ ppserver -a -p 35000

Then, locally, instantiate a ``Server`` with the connection information
for the remote node, submit some jobs, and retrieve the results::

    >>> job_server = pp.Server(ppservers=('127.0.0.1:35000',))
    >>> f1 = job_server.submit(math.sin, (math.pi/2,), (), ('math',))
    >>> f2 = job_server.submit(math.sin, (0,), (), ('math',))
    >>> f3 = job_server.submit(math.sin, (-math.pi/2,), (), ('math',))
    >>> f1(),f2(),f3()
    (1.0, 0.0, -1.0)
    >>> 

However, the stats show that all of the jobs were run locally::

    >>> job_server.print_stats()
    Job execution statistics:
     job count | % of all jobs | job time sum | time per job | job server
             3 |        100.00 |       0.0024 |     0.000812 | local
    Time elapsed since server creation 31.755322217941284
    0 active tasks, 4 cores

This is due because we don't specify the number of nodes. The number of nodes
are specified both in the ``ppserver`` and in the local job ``Server``. Thus,
the above is actually "autobalance" between 4 local nodes and 4 remote nodes.
The former is naturally going to be preferred; however, if the local server is
flooded with jobs, some will get sent to the remote ``ppserver``, and that will
be reflected in the stats.  To run all jobs remotely, set the number of local
nodes to zero::

    >>> job_server = pp.Server(0, ppservers=('127.0.0.1:35000',))
    >>> f1 = job_server.submit(math.sin, (math.pi/2,), (), ('math',))
    >>> f2 = job_server.submit(math.sin, (0,), (), ('math',))
    >>> f3 = job_server.submit(math.sin, (-math.pi/2,), (), ('math',))
    >>> f1(),f2(),f3()
    (1.0, 0.0, -1.0)
    >>> job_server.print_stats()
    Job execution statistics:
     job count | % of all jobs | job time sum | time per job | job server
             3 |        100.00 |       0.0016 |     0.000518 | 127.0.0.1:35000
    Time elapsed since server creation 15.123202800750732
    0 active tasks, 0 cores

>>> 

Get help on the command line options for ``ppserver``::

    $ ppserver --help
    Parallel Python Network Server (pp-1.7.6.9)
    Usage: ppserver [-hdar] [-f format] [-n proto] [-c config_path] [-i interface] [-b broadcast] [-p port] [-w nworkers] [-s secret] [-t seconds] [-k seconds] [-P pid_file]

    Options: 
    -h                 : this help message
    -d                 : set log level to debug
    -f format          : log format
    -a                 : enable auto-discovery service
    -r                 : restart worker process after each task completion
    -n proto           : protocol number for pickle module
    -c path            : path to config file
    -i interface       : interface to listen
    -b broadcast       : broadcast address for auto-discovery service
    -p port            : port to listen
    -w nworkers        : number of workers to start
    -s secret          : secret for authentication
    -t seconds         : timeout to exit if no connections with clients exist
    -k seconds         : socket timeout in seconds
    -P pid_file        : file to write PID to

    To print server stats send SIGUSR1 to its main process (unix only). 

    Due to the security concerns always use a non-trivial secret key.
    Secret key set by -s switch will override secret key assigned by
    pp_secret variable in .pythonrc.py


More Information
----------------
Probably the best way to get started is to look at the documentation at
http://ppft.rtfd.io. Also, you can see a set of example scripts in
``ppft.tests``. You can run the test suite with ``python -m ppft.tests``.
``ppft`` will create and execute jobs on local workers (automatically created
using ``python -u -m ppft``). Additionally, remote servers can be created with 
``ppserver`` (or ``python -m ppft.server``), and then jobs can be distributed
to remote workers. See ``--help`` for more details on how to configure a server.
Please feel free to submit a ticket on github, or ask a question on
stackoverflow (**@Mike McKerns**).  If you would like to share how you use
``ppft`` in your work, please send an email (to **mmckerns at uqfoundation dot org**).


Citation
--------
If you use ``ppft`` to do research that leads to publication, we ask that you
acknowledge use of ``ppft`` by citing the following in your publication::

    M.M. McKerns, L. Strand, T. Sullivan, A. Fang, M.A.G. Aivazis,
    "Building a framework for predictive science", Proceedings of
    the 10th Python in Science Conference, 2011;
    http://arxiv.org/pdf/1202.1056

    Michael McKerns and Michael Aivazis,
    "pathos: a framework for heterogeneous computing", 2010- ;
    https://uqfoundation.github.io/project/pathos

Please see https://uqfoundation.github.io/project/pathos or
http://arxiv.org/pdf/1202.1056 for further information.

