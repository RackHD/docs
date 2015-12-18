Logging in RacKHD
--------------------------------

Log Levels
~~~~~~~~~~~

We have a common set of logging levels within RackHD, used across the projects
and applications. The levels are defined in the `on-core library`_

.. _on-core library: https://github.com/RackHD/on-core/blob/master/lib/common/constants.js#L36

The conventions for using the levels are:

critical
  Used for logging terminal failures that are crashing the system, for
  information to support post-failure debugging. Errors logged as critical are
  expected to be terminal and will likely result in the application crashing or
  failing to start.

  Errors logged at a **critical** level should be actionable in that the
  tracebacks or logged errors should allow resolution of the error with a code
  or configuration update. These errors are generally considered failures of
  the program to anticipate corner conditions or failure modes.

error
  Logging errors that may (or will) result in the application behaving in an
  unexpected fashion. Assertion/precondition errors are appropriate here, as
  well as any error that would generate an "unknown" error and be exposed via
  a 500 response (i.e. an undefined error) in an HTTP response code. The results
  of these errors are not expected to be terminal to the operation of the
  application.

  Errors logged at an **error** level should be actionable in that the
  tracebacks or logged errors should allow resolution of the error with a code
  or configuration update. These errors are generally considered failures of
  the program to anticipate corner conditions or failure modes.

warning
  An expected error condition or fault in inputs to which the application responds
  correctly, but the end-user action may not be what they intended. Incorrect
  passwords, or actions that are not allowed because they conflict with existing
  configurations are appropriate for this level.

  Errors logged at an **warning** level may not be actionable, but should be
  informative in the logs to indicate what the failure was. Errors where secure
  information are part of the response may include more information in logs than
  in a response ot the end user for security considerations.


info
  Informational data about current execution that would be relevant to regular
  use of the application. Not generally considered "errors" at the log level
  of **info**, this level should be used judiciously with the idea that regular
  operation of the application is likely to run with log filtering set to allow
  **info** logging.

  Information logged at the **info** is not expected to be actionable, but may
  be expected to be used in external systems collecting the log information for
  regular operational metrics.

debug
  Informational data about current execution that would be relevant to debugging
  or detailed analysis of the application, typically for a programmer, or to
  generate logs for post-analysis by a someone familiar with the code in the
  project. Information is not considered "errors" at the log level of **debug**.

  Information logged at the **debug** is not expected to be actionable, but may
  be expected to be used in external systems collecting the log information for
  debugging or post-analysis metrics.

Setting up and using Logging
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using our dependency injection libraries, it's typical to inject ``Logger`` and
then use it within appropriate methods. Within factory methods for services or
modules, ``Logger`` is initialized wiht the module name, which annotates the
logs with information about where the logs were coming from.

An example of this::

    di.annotate(someFactory, new di.Inject('Logger'))

    function someFactory (Logger) {
        var logger = Logger.initialize(someFactory);
    }

with ``logger`` being used later within the relevant scope for logging. For
example::

    function foo(bar, baz) {
        logger.debug("Another request was made with ", {id: baz});
    }

The definitions for the methods and what the code does can be found in the
`logger module`_.

.. _logger module: https://github.com/RackHD/on-core/blob/master/lib/common/logger.js

Deprecation
~~~~~~~~~~~~

There is a special function in our logging common library for including in methods
you're attempting to deprecate::

    logger.deprecate("This shouldn't be used any longer", 2)

Which will generate log output at the **error** for assistance in identifying
methods, APIs, or subsystems that are still in use but in the process of being
depracted for replacement.
