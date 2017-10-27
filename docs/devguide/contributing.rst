Contributing Changes to RackHD
--------------------------------

Guidelines for merging pull requests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For code changes, we currently use a guideline of `lazy consensus`_  with two
positive reviews with at least one of those reviews being one of the core
maintainers and no negative votes. And of course, the gates for the pull
requests must pass as well (unit tests, etc).

If you put a review up, please be explicit with a vote (+1, -1, or +/-0) so
we can distinguish questions asking for information or background from reviews
implying that the relevant change should not be merged. Likewise if you put up
a change for review as a pull request, a -1 review comment isn’t a reflection
on you as a person, instead is a request to make a modification before that pull
request should be merged.

.. _lazy consensus: http://www.apache.org/foundation/glossary.html#LazyConsensus

**For those with commit privileges**

See https://github.com/RackHD/RackHD/wiki/Merge-Guidelines for more informal guidelines
and rules of thumb to follow when making merge decisions.


Getting commit privileges
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The core committer team will grant contributor rights to the RackHD project
using a `lazy consensus`_ mechanism. Any of the maintainers/core contributors
can nominate someone to have those privileges, and with two +1 votes and no
negative votes, the team will grant commit privileges.

The core team will also be responsible for removing commit privileges when
appropriate - for example for malicious merge behavior or just inactivity over
an extended period of time.


Quality gates for the pull requests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are three quality gates to ensure the pull requests quality, `Hound`_ for
code style check, `Travis CI`_ for unit-test and coveralls, `Jenkins`_ for the combination
test including unit-test and smoke test. When a pull request is created, all tests
will run automatically, and the test results can be found in the merge status field of
each pull request page.
Running unit/functional tests locally prior to creating a pull request is strongly encouraged.
This would hopefully minimize the amount errors seen during PR submission and lessen a
dependency on Travis/Jenkins to test code before it's really ready to be submitted.

.. _Hound: https://houndci.com/
.. _Travis CI: https://travis-ci.org/
.. _Jenkins: https://jenkins.io/
.. _jshint: http://jshint.com/

**Hound**

Hound works with `jshint`_ and comments on style violations in pull requests.
Configuration files ``.hound.yml`` and ``.jshintrc`` have been created in each
repository, so before creating a pull request, you can check code style locally with
jshint to find out style violations beforehand.

**Travis CI**

Travis CI runs the unit tests, and then does some potentially ancillary actions.
The build specifics are detailed in the ``.travis.yml`` file within each repository.
For finding out basic errors before creating a pull request, you can run unit test
locally using ``npm test`` within each repository.

**Concourse**

RackHD uses Concourse CI to monitor and perform quality gate tests on all pull requests
prior to merge. The gates include running all the unit tests, running all dependent
project unit tests with the code proposed from the pull request, running an integration
"smoke test" to verify basic end to end functionality and commenting on the details of
test case failure. Concourse can also take instructions from pull request comments or
description in order to handle more complex test scenarios.  Instructions can be written
in the pull request description or comments.

All pull requests will need to be labeled with the "run-test" label before the quality
gate tests will run.  This label needs to be set by a RackHD Commit.

The following table show all the Jenkins Instructions and usage:

.. list-table::
    :widths: 30 50 100
    :header-rows: 1

    * - Instruction
      - Description
      - Detailed Usage
    * - depends on: pr1_url
        depends on: pr2_url
        ...
      - Trigger one Jenkins test that using the commits of all interdependent pull requests.
      - RackHD is a multi repository project, so there are times one new feature needs
        changes on two or more repositories. In such situation neither Concourse test for single
        pull request can pass. This command is order to solve this problem.

        Recommended usage: for interdependent pull requests, first create pull request one by one, but
        do not label any PRs with "run-test".  When creating the last pull request include the depend
        statements in the description:

        .. code::

            depends on: pr1_url
            depends on: pr2_url
            ...

        Then set the "run-test" label only on the pull request that includes the depends on instruction.

        The interdependent test result will be written back to all interdependent pull requests. The unit test
        error log will be commented on each related pull request, the functional test error log will only be
        commented on the main pull request, the one with the "depends on ..." instruction.
