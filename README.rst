wrapt_timeout_decorator
=======================

|Pypi Status| |license| |maintenance| |jupyter|

|Build Status| |Codecov Status| |Better Code| |code climate| |snyk security|

.. |license| image:: https://img.shields.io/github/license/webcomics/pywine.svg
   :target: http://en.wikipedia.org/wiki/MIT_License
.. |maintenance| image:: https://img.shields.io/maintenance/yes/2019.svg
.. |Build Status| image:: https://travis-ci.org/bitranox/wrapt_timeout_decorator.svg?branch=master
   :target: https://travis-ci.org/bitranox/wrapt_timeout_decorator
.. for the pypi status link note the dashes, not the underscore !
.. |Pypi Status| image:: https://badge.fury.io/py/wrapt-timeout-decorator.svg
   :target: https://badge.fury.io/py/wrapt_timeout_decorator
.. |Codecov Status| image:: https://codecov.io/gh/bitranox/wrapt_timeout_decorator/branch/master/graph/badge.svg
   :target: https://codecov.io/gh/bitranox/wrapt_timeout_decorator
.. |Better Code| image:: https://bettercodehub.com/edge/badge/bitranox/wrapt_timeout_decorator?branch=master
   :target: https://bettercodehub.com/results/bitranox/wrapt_timeout_decorator
.. |snyk security| image:: https://snyk.io/test/github/bitranox/wrapt_timeout_decorator/badge.svg
   :target: https://snyk.io/test/github/bitranox/wrapt_timeout_decorator
.. |jupyter| image:: https://mybinder.org/badge.svg
   :target: https://mybinder.org/v2/gh/bitranox/wrapt_timeout_decorator/master?filepath=jupyter_test_wrapt_timeout_decorator.ipynb
.. |code climate| image:: https://api.codeclimate.com/v1/badges/2b2b6589f80589689c2b/maintainability
   :target: https://codeclimate.com/github/bitranox/wrapt_timeout_decorator/maintainability
   :alt: Maintainability

there are many timeout decorators out there - that one focuses on correctness when using with Classes, methods,
class methods, static methods and so on, preserving also the traceback information for Pycharm debugging.

There is also a powerful eval function, it allows to read the desired timeout value even from Class attributes.

It is very flexible and can be used with python >= 3.5, pypy3 and probably other dialects.

There are two timeout strategies implemented, the ubiquitous method using "Signals" and the second using Multiprocessing.
Using "Signals" is slick and lean, but there are nasty caveats, please check section `Caveats using Signals`_

The default strategy is therefore using Multiprocessing, but You can also use Signals, You have been warned !

Due to the lack of signals on Windows, or for threaded functions (in a subthread) where signals cant be used, Your only choice is Multiprocessing,
this is set automatically.

Under Windows the decorated function and results needs to be pickable.
For that purpose we use "multiprocess" and "dill" instead of "multiprocessing" and "pickle", in order to be able to use this decorator on more sophisticated objects.
Communication to the subprocess is done via "multiprocess.pipe" instead of "queue", which is faster and might also work on Amazon AWS.

`100% code coverage <https://codecov.io/gh/bitranox/wrapt_timeout_decorator>`_, mypy static type checking, tested under `Linux, OsX, Windows and Wine <https://travis-ci.org/bitranox/wrapt_timeout_decorator>`_, automatic daily builds  and monitoring

----

- `Try it Online`_
- `Installation and Upgrade`_
- `Basic Usage`_
- `use with Windows`_
- `Caveats using Signals`_
- `nested Timeouts`_
- `Alternative Exception`_
- `Parameters`_
- `Override Parameters`_
- `Multithreading`_
- `use as function not as decorator`_
- `use powerful eval function`_
- `detect pickle errors`_
- `Logging in decorated functions`_
- `hard timeout`_ or - when the timout should start ?
- `Requirements`_
- `Acknowledgements`_
- `Contribute`_
- `Report Issues <https://github.com/bitranox/wrapt_timeout_decorator/blob/master/ISSUE_TEMPLATE.md>`_
- `Pull Request <https://github.com/bitranox/wrapt_timeout_decorator/blob/master/PULL_REQUEST_TEMPLATE.md>`_
- `Code of Conduct <https://github.com/bitranox/wrapt_timeout_decorator/blob/master/CODE_OF_CONDUCT.md>`_
- `License`_
- `Changelog`_

----

Try it Online
-------------

You might try it right away in Jupyter Notebook by using the "launch binder" badge, or click `here <https://mybinder.org/v2/gh/bitranox/wrapt_timeout_decorator/master?filepath=jupyter_test_wrapt_timeout_decorator.ipynb>`_

Installation and Upgrade
------------------------

From source code:

.. code-block:: bash

    # normal install
    python setup.py install
    # test without installing
    python setup.py test

via pip latest Release:

.. code-block:: bash

    # latest Release from pypi
    pip install wrapt_timeout_decorator

    # test without installing
    pip install wrapt_timeout_decorator --install-option test

via pip latest Development Version:

.. code-block:: bash

    # upgrade all dependencies regardless of version number (PREFERRED)
    pip install --upgrade git+https://github.com/bitranox/wrapt_timeout_decorator.git --upgrade-strategy eager
    # normal install
    pip install --upgrade git+https://github.com/bitranox/wrapt_timeout_decorator.git
    # test without installing
    pip install git+https://github.com/bitranox/wrapt_timeout_decorator.git --install-option test

via requirements.txt:

.. code-block:: bash

    # Insert following line in Your requirements.txt:
    # for the latest Release:
    wrapt_timeout_decorator
    # for the latest Development Version :
    git+https://github.com/bitranox/wrapt_timeout_decorator.git

    # to install and upgrade all modules mentioned in requirements.txt:
    pip install --upgrade -r /<path>/requirements.txt

via python:

.. code-block:: python

    # for the latest Release
    python -m pip install upgrade wrapt_timeout_decorator

    # for the latest Development Version
    python -m pip install upgrade git+https://github.com/bitranox/wrapt_timeout_decorator.git

Basic Usage
-----------

.. code-block:: py

    import time
    from wrapt_timeout_decorator import *

    @timeout(5)
    def mytest(message):
        # this example does NOT work on windows, please check the section
        # "use with Windows" in the README.rst
        print(message)
        for i in range(1,10):
            time.sleep(1)
            print('{} seconds have passed'.format(i))

    if __name__ == '__main__':
        mytest('starting')

use with Windows
----------------

For the impatient:

All You need to do is to put the decorated function into another Module, NOT in the main program.

For those who want to dive deeper :


On Windows the main module is imported again (but with a name != 'main') because Python is trying to simulate
a forking-like behavior on a system that doesn't support forking. multiprocessing tries to create an environment
similar to Your main process by importing the main module again with a different name. Thats why You need to shield
the entry point of Your program with the famous " if __name__ == '__main__': "

.. code-block:: py

    import lib_foo

    def some_module():
        lib_foo.function_foo()

    def main():
        some_module()


    # here the subprocess stops loading, because __name__ is NOT '__main__'
    if __name__ = '__main__':
        main()

This is a problem of Windows OS, because the Windows Operating System does not support "fork"

You can find more information on that here:

https://stackoverflow.com/questions/45110287/workaround-for-using-name-main-in-python-multiprocessing

https://docs.python.org/2/library/multiprocessing.html#windows

Since main.py is loaded again with a different name but "__main__", the decorated function now points to objects that do not exist anymore, therefore You need to put the decorated Classes and functions into another module.
In general (especially on windows) , the main() program should not have anything but the main function, the real thing should happen in the modules.
I am also used to put all settings or configurations in a different file - so all processes or threads can access them (and also to keep them in one place together, not to forget typing hints and name completion in Your favorite editor)

The "dill" serializer is able to serialize also the __main__ context, that means the objects in our example are pickled to "__main__.lib_foo", "__main__.some_module","__main__.main" etc.
We would not have this limitation when using "pickle" with the downside that "pickle" can not serialize following types:

functions with yields, nested functions, lambdas, cell, method, unboundmethod, module, code, methodwrapper,
dictproxy, methoddescriptor, getsetdescriptor, memberdescriptor, wrapperdescriptor, xrange, slice,
notimplemented, ellipsis, quit

additional dill supports:

save and load python interpreter sessions, save and extract the source code from functions and classes, interactively diagnose pickling errors

To support more types with the decorator, we selected dill as serializer, with the small downside that methods and classes can not be decorated in the __main__ context, but need to reside in a module.

You can find more information on that here:
https://stackoverflow.com/questions/45616584/serializing-an-object-in-main-with-pickle-or-dill

**Timing :** Since spawning takes some unknown timespan (all imports needs to be done again !), You can specify when the timeout should start, please read the section `hard timeout`_

Here an example that will work on Linux but wont work on Windows (the variable "name" and the function "sleep" wont be found in the spawned process :


.. code-block:: py

    main.py:

    from time import sleep
    from wrapt_timeout_decorator import *

    name="my_var_name"

    @timeout(5, use_signals=False)
    def mytest():
        # this example does NOT work on windows, please check the example below !
        # You need to move this function into a module to be able to run it on windows.
        print("Start ", name)
        for i in range(1,10):
            sleep(1)
            print("{} seconds have passed".format(i))
        return i


    if __name__ == '__main__':
        mytest()


here the same example, which will work on Windows:


.. code-block:: py


    # my_program_main.py:

    import lib_test

    def main():
        lib_test.mytest()

    if __name__ == '__main__':
        main()


.. code-block:: py


        # conf_my_program.py:

        class ConfMyProgram(object):
            def __init__(self):
                self.name:str = 'my_var_name'

        conf_my_program = ConfMyProgram()


.. code-block:: py

    # lib_test.py:

    from wrapt_timeout_decorator import *
    from time import sleep
    from conf_my_program import conf_my_program

    # use_signals = False is not really necessary here, it is set automatically under Windows
    # but You can force NOT to use Signals under Linux
    @timeout(5, use_signals=False)
    def mytest():
        print("Start ", conf_my_program.name)
        for i in range(1,10):
            sleep(1)
            print("{} seconds have passed".format(i))
        return i

Caveats using Signals
---------------------

as ABADGER1999 points out in his blog https://anonbadger.wordpress.com/2018/12/15/python-signal-handlers-and-exceptions/
using signals and the TimeoutException is probably not the best idea - because it can be catched in the decorated function.

Of course You can use Your own Exception, derived from the Base Exception Class, but the code might still not work as expected -
see the next example - You may try it out in `jupyter <https://mybinder.org/v2/gh/bitranox/wrapt_timeout_decorator/master?filepath=jupyter_test_wrapt_timeout_decorator.ipynb>`_:

.. code-block:: py

    import time
    from wrapt_timeout_decorator import *

    # caveats when using signals - the TimeoutError raised by the signal may be catched
    # inside the decorated function.
    # So You might use Your own Exception, derived from the base Exception Class.
    # In Python-3.7.1 stdlib there are over 300 pieces of code that will catch your timeout
    # if you were to base an exception on Exception. If you base your exception on BaseException,
    # there are still 231 places that can potentially catch your exception.
    # You should use use_signals=False if You want to make sure that the timeout is handled correctly !
    # therefore the default value for use_signals = False on this decorator !

    @timeout(5, use_signals=True)
    def mytest(message):
        try:
            print(message)
            for i in range(1,10):
                time.sleep(1)
                print('{} seconds have passed - lets assume we read a big file here'.format(i))
        # TimeoutError is a Subclass of OSError - therefore it is catched here !
        except OSError:
            for i in range(1,10):
                time.sleep(1)
                print('Whats going on here ? - Ooops the Timeout Exception is catched by the OSError ! {}'.format(i))
        except Exception:
            # even worse !
            pass
        except:
            # the worst - and exists more then 300x in actual Python 3.7 stdlib Code !
            # so You never really can rely that You catch the TimeoutError when using Signals !
            pass


    if __name__ == '__main__':
        try:
            mytest('starting')
            print('no Timeout Occured')
        except TimeoutError():
            # this will never be printed because the decorated function catches implicitly the TimeoutError !
            print('Timeout Occured')

nested Timeouts
----------------

since there is only ONE ALARM Signal on Unix per process, You need to use use_signals = False for nested timeouts.
The outmost decorator might use Signals, all nested Decorators needs to use use_signals=False (the default)
You may try it out in `jupyter <https://mybinder.org/v2/gh/bitranox/wrapt_timeout_decorator/master?filepath=jupyter_test_wrapt_timeout_decorator.ipynb>`_:

.. code-block:: py

    # main.py
    import mylib

    # this example will work on Windows and Linux
    # since the decorated function is not in the __main__ scope but in another module !

    if __name__ == '__main__':
    mylib.outer()


.. code-block:: py

    # mylib.py
    from wrapt_timeout_decorator import *
    import time

    # this example will work on Windows and Linux
    # since the decorated function is not in the __main__ scope but in another module !

    @timeout(1, use_signals=True)
    def outer():
        inner()

    @timeout(5)
    def inner():
        time.sleep(3)
        print("Should never be printed if you call outer()")

Alternative Exception
---------------------

Specify an alternate exception to raise on timeout:

.. code-block:: py

    import time
    from wrapt_timeout_decorator import *

    @timeout(5, timeout_exception=StopIteration)
    def mytest(message):
        # this example does NOT work on windows, please check the section
        # "use with Windows" in the README.rst
        print(message)
        for i in range(1,10):
            time.sleep(1)
            print('{} seconds have passed'.format(i))

    if __name__ == '__main__':
        mytest('starting')

Parameters
----------

.. code-block:: py

    @timeout(dec_timeout, use_signals, timeout_exception, exception_message, dec_allow_eval, dec_hard_timeout)
    def decorated_function(*args, **kwargs):
        # interesting things happens here ...
        ...

    """
    dec_timeout         the timeout period in seconds, or a string that can be evaluated when dec_allow_eval = True
                        type: float, integer or string
                        default: None (no Timeout set)
                        can be overridden by passing the kwarg dec_timeout to the decorated function*

    use_signals         if to use signals (linux, osx) to realize the timeout. The most accurate method but with caveats.
                        By default the Wrapt Timeout Decorator does NOT use signals !
                        Please note that signals can only be used in the main thread and only on linux. In all other cases
                        (not the main thread, or under Windows) signals cant be used anyway and will be disabled automatically.
                        In general You dont need to set use_signals Yourself. Please read the section - `Caveats using Signals`_
                        type: boolean
                        default: False
                        can be overridden by passing the kwarg use_signals to the decorated function*

    timeout_exception   the Exception that will be raised if a timeout occurs.
                        type: exception
                        default: TimeoutError, on Python < 3.3: Assertion Error (since TimeoutError does not exist on that Python Versions)

    exception_message   custom Exception message.
                        type: str
                        default : 'Function {function_name} timed out after {dec_timeout} seconds' (will be formatted)

    dec_allow_eval      will allow to evaluate the parameter dec_timeout.
                        If enabled, the parameter of the function dec_timeout, or the parameter passed
                        by kwarg dec_timeout will be evaluated if its type is string. You can access :
                        wrapped (the decorated function object and all the exposed objects below)
                        instance    Example: 'instance.x' - see example above or doku
                        args        Example: 'args[0]' - the timeout is the first argument in args
                        kwargs      Example: 'kwargs["max_time"] * 2'
                        type: bool
                        default: false
                        can be overridden by passing the kwarg dec_allow_eval to the decorated function*

    dec_hard_timeout    only relevant when signals can not be used. In that case a new process needs to be created.
                        The creation of the process on windows might take 0.5 seconds and more, depending on the size
                        of the main module and modules to be imported. Especially useful for small timeout periods.

                        dec_hard_timeout = True : the decorated function will time out after dec_timeout, no matter what -
                        that means if You set 0.1 seconds here, the subprocess can not be created in that time and the
                        function will always time out and never run.

                        dec_hard_timeout = False : the decorated function will time out after the called function
                        is allowed to run for dec_timeout seconds. The time needed to create that process is not considered.
                        That means if You set 0.1 seconds here, and the time to create the subprocess is 0.5 seconds,
                        the decorated function will time out after 0.6 seconds in total, allowing the decorated function to run
                        for 0.1 seconds.

                        type: bool
                        default: false
                        can be overridden by passing the kwarg dec_hard_timeout to the decorated function*

    * that means the decorated_function must not use that kwarg itself, since this kwarg will be popped from the kwargs
    """

Override Parameters
-------------------

decorator parameters starting with \dec_* and use_signals can be overridden by kwargs with the same name :

.. code-block:: py


    import time
    from wrapt_timeout_decorator import *

    @timeout(dec_timeout=5, use_signals=False)
    def mytest(message):
        # this example does NOT work on windows, please check the section
        # "use with Windows" in the README.rst
        print(message)
        for i in range(1,10):
            time.sleep(1)
            print('{} seconds have passed'.format(i))

    if __name__ == '__main__':
        mytest('starting',dec_timeout=12)   # override the decorators setting. The kwarg dec_timeout will be not
                                            # passed to the decorated function.

Multithreading
--------------

By default, timeout-decorator uses signals to limit the execution time
of the given function. This approach does not work if your function is
executed not in the main thread (for example if it's a worker thread of
the web application) or when the operating system does not support signals (aka Windows).
There is an alternative timeout strategy for this case - by using multiprocessing.
This is done automatically, so you dont need to set ``use_signals=False``.
You can force not to use signals on Linux by passing the parameter ``use_signals=False`` to the timeout
decorator function for testing. If Your program should (also) run on Windows, I recommend to test under
Windows, since Windows does not support forking (read more under Section ``use with Windows``).
The following Code will run on Linux but NOT on Windows :

.. code-block:: py

    import time
    from wrapt_timeout_decorator import *

    @timeout(5, use_signals=False)
    def mytest(message):
        # this example does NOT work on windows, please check the section
        # "use with Windows" in the README.rst
        print(message)
        for i in range(1,10):
            time.sleep(1)
            print('{} seconds have passed'.format(i))

    if __name__ == '__main__':
        mytest('starting')

.. warning::
    Make sure that in case of multiprocessing strategy for timeout, your function does not return objects which cannot
    be pickled, otherwise it will fail at marshalling it between master and child processes. To cover more cases,
    we use multiprocess and dill instead of multiprocessing and pickle.

    Since Signals will not work on Windows, it is disabled by default, whatever You set.

use as function not as decorator
--------------------------------

You can use the timout also as function, without using as decorator:

.. code-block:: py

    import time
    from wrapt_timeout_decorator import *

    def mytest(message):
        print(message)
        for i in range(1,10):
            time.sleep(1)
            print('{} seconds have passed'.format(i))

    if __name__ == '__main__':
        timeout(dec_timeout=5)(mytest)('starting')

use powerful eval function
--------------------------

This is very powerful, but can be also very dangerous if you accept strings to evaluate from UNTRUSTED input.

read: https://nedbatchelder.com/blog/201206/eval_really_is_dangerous.html

If enabled, the parameter of the function dec_timeout, or the parameter passed by kwarg dec_timeout will
be evaluated if its type is string.

You can access :

- "wrapped"
   (the decorated function and his attributes)

- "instance"
   Example: 'instance.x' - an attribute of the instance of the class instance

- "args"
   Example: 'args[0]' - the timeout is the first argument in args

- "kwargs"
   Example: 'kwargs["max_time"] * 2'

- and of course all attributes You can think of - that makes it powerful but dangerous.
   by default allow_eval is disabled - but You can enable it in order to cover some edge cases without
   modifying the timeout decorator.


.. code-block:: py

    # this example does NOT work on windows, please check the section
    # "use with Windows" in the README.rst
    def class ClassTest4(object):
        def __init__(self,x):
            self.x=x

        @timeout('instance.x', dec_allow_eval=True)
        def test_method(self):
            print('swallow')

        @timeout(1)
        def foo3(self):
            print('parrot')

        @timeout(dec_timeout='args[0] + kwargs.pop("more_time",0)', dec_allow_eval=True)
        def foo4(self,base_delay):
            time.sleep(base_delay)
            print('knight')


    if __name__ == '__main__':
        # or override via kwarg :
        my_foo = ClassTest4(3)
        my_foo.test_method(dec_timeout='instance.x * 2.5 +1')
        my_foo.foo3(dec_timeout='instance.x * 2.5 +1', dec_allow_eval=True)
        my_foo.foo4(1,more_time=3)  # this will time out in 4 seconds

detect pickle errors
--------------------

remember that decorated functions (and their results !) needs to be pickable under Windows. In order to detect pickle problems You can use :

.. code-block:: py

    from wrapt_timeout_decorator import *
    # always remember that the "object_to_pickle" should not be defined within the main context
    detect_unpickable_objects(object_to_pickle, dill_trace=True)  # type: (Any, bool) -> Dict

Logging in decorated functions
------------------------------

when signals=False (on Windows), logging in the wrapped function can be tricky. Since a new process is
created, we can not use the logger object of the main process. Further development is needed to
connect to the main process logger via a socket or queue.

When the wrapped function is using logger=logging.getLogger(), a new Logger Object is created.
Setting up that Logger can be tricky (File Logging from two Processes is not supported ...)
I think I will use a socket to implement that (SocketHandler and some Receiver Thread)

Until then, You need to set up Your own new logger in the decorated function, if logging is needed.
Again - keep in mind that You can not write to the same logfile from different processes !
(although there are logging modules which can do that)

hard timeout
------------

when use_signals = False (this is the only method available on Windows), the timeout function is realized by starting
another process and terminate that process after the given timeout.
Under Linux fork() of a new process is very fast, under Windows it might take some considerable time,
because the main context needs to be reloaded on spawn().
Spawning of a small module might take something like 0.5 seconds and more.

By default, when using signals=False, the timeout begins after the new process is created.

This means that the timeout given, is the time the decorated process is allowed to run, not included the time excluding the time to setup the process itself.
This is especially important if You use small timeout periods :

for Instance:


.. code-block:: py

    @timeout(0.1)
    def test():
        time.sleep(0.2)


the total time to timeout on linux with use_signals = False will be around 0.1 seconds, but on windows this can take
about 0.6 seconds: 0.5 seconds to spawn the new process, and giving the function test() 0.1 seconds to run !

If You need that a decorated function should timeout exactly** after the given timeout period, You can pass
the parameter dec_hard_timeout=True. in this case the called function will timeout exactly** after the given time,
no matter how long it took to spawn the process itself. In that case, if You set up the timeout too short,
the process might never run and will always timeout during spawning.

** well, more or less exactly - it still takes some short time to return from the spawned process - so be extra cautious on very short timeouts !

Requirements
------------

following modules will be automatically installed :

.. code-block:: shell

    ## Test Requirements
    ## following Requirements will be installed temporarily for
    ## "setup.py install test" or "pip install <package> --install-option test"
    typing ; python_version < "3.5"
    pathlib; python_version < "3.4"
    mypy ; platform_python_implementation != "PyPy" and python_version >= "3.5"
    pytest
    pytest-pep8 ; python_version < "3.5"
    pytest-codestyle ; python_version >= "3.5"
    pytest-mypy ; platform_python_implementation != "PyPy" and python_version >= "3.5"
    pytest-runner

    ## Project Requirements
    dill
    multiprocess
    wrapt

Acknowledgements
----------------

Derived from

https://github.com/pnpnpn/timeout-decorator

http://www.saltycrane.com/blog/2010/04/using-python-timeout-decorator-uploading-s3/

thanks to abadger1999 for pointing out caveats when using signals, see :
https://anonbadger.wordpress.com/2018/12/15/python-signal-handlers-and-exceptions/

special thanks to "uncle bob" Robert C. Martin, especially for his books on "clean code" and "clean architecture"

Contribute
----------

I would love for you to fork and send me pull request for this project.
- `please Contribute <https://github.com/bitranox/wrapt_timeout_decorator/blob/master/CONTRIBUTING.md>`_

License
-------

This software is licensed under the `MIT license <http://en.wikipedia.org/wiki/MIT_License>`_

-----------------------------------------------------------------

.. Changelog link comes from the included document !

Changelog
=========

1.3.1
-----
2019-09-02: strict mypy static type checking, housekeeping

1.3.0
-----
2019-05-03: pointing out caveats when using signals, the decorator defaults now to NOT using Signals !

1.2.9
-----
2019-05-03: support nested decorators, mypy static type checking

1.2.8
-----
2019-04-23: import multiprocess as multiprocess, not as multiprocessing - that might brake other packages

1.2.0
------
2019-04-09: initial PyPi release

1.1.0
-----
2019-04-03: added pickle analyze convenience function

1.0.9
-----
2019-03-27: added OsX and Windows tests, added parameter dec_hard_timeout for Windows, 100% Code Coverage

1.0.8
-----
2019-02-26: complete refractoring and code cleaning

1.0.7
-----
2019-02-25:  fix pickle detection, added some tests, codecov now correctly combining the coverage of all tests

1.0.6
-----
2019-02-24: fix pickle detection when use_signals = False, drop Python2.6 support since wrapt dropped it.

1.0.5
-----
2018-09-13: use multiprocessing.pipe instead of queue
If we are not able to use signals, we need to spawn a new process.
This was done in the past by pickling the target function and put it on a queue -
now this is done with a half-duplex pipe.

- it is faster
- it probably can work on Amazon AWS, since there You must not use queues

1.0.4
-----

2017-12-02: automatic detection if we are in the main thread. Signals can only be used in the main thread. If the decorator is running in a subthread, we automatically disable signals.


1.0.3
-----

2017-11-30: using dill and multiprocess to enhance windows functionality


1.0.0
-----

2017-11-10: Initial public release

