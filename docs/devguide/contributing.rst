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
code style check, `Travis CI`_ for unit-test, `Jenkins`_ for re-unit-test and
functional test. When a pull request created, all tests will run automatically, test 
results can be found in the merge status field of each pull request page.

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
locally using scripts ``HWIMO-TEST`` within each repository.

**Jenkins**

Jenkins uses the Github Pull Request Builder plugin to monitoring all pull requests
to perform quality gate tests prior to merge. The gates include running all the unit
tests, running all dependent project unit tests with the code proposed from the pull 
request, running an integration "smoke test" to verify basic end to end functionality
and commenting on the details of test case failure. Jenkins is also able to takes 
instructions from pull request comments to deal with complex test scenarios. 
Instructions can be written in pull request description or comments.

The following table show all the Jenkins Instructions and usage:

.. list-table::
    :widths: 30 50 100 
    :header-rows: 1

    * - Command
      - Description
      - Detailed Usage
    * - Jenkins: test this please
      - Trigger one Jenkins test on current pull request.
      - This command will trigger one Jenkins test whether the pull request has been
        tested or not, the new test result will cover the previous one. 
    * - Jenkins: ignore
      - Avoid running any test in the next Jenkins test.
      - This command can be used in "work in progress" pull request or one of 
        "interdependent pull requests" to avoid a Jenkins test that is destined to fail.
        The commit status of this pull request will be set to "pending" temporally.
        Jenkins periodically check the repository status so to guarantee to avoid the 
        unexpected test, it's strongly advised to write this command in pull request description.
        Otherwise if write in comments, an unexpected test may be triggered in the time slot
        between pull request creation and writing up comments. 
    * - Jenkins: depends on pr1_url, pr2_url ...
      - Trigger one Jenkins test that using the commits of all interdependent pull requests.
      - RackHD is a multi repositories project, so there are times one new feature leads to 
        changes on two or more repositories. In such situation neither Jenkins test for single
        pull request can pass. This command is order to solve this problem. 

        Recommended usage: for interdependent pull requests, first create pull request one by one
        and write "jenkins: ignore" in description of each pull request until the last one. When
        create the last pull request write "jenkins: depends on pr1_url, pr2_url ..." in its description.

        Under this usage duplicated or unexpected Jenkins test can be avoided.

