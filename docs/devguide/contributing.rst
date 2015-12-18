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


Getting commit privileges
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The core committer team will grant contributor rights to the RackHD project
using a `lazy consensus`_ mechanism. Any of the maintainers/core contributors
can nominate someone to have those privileges, and with two +1 votes and no
negative votes, the team will grant commit privileges.

The core team will also be responsible for removing commit privileges when
appropriate - for example for malicious merge behavior or just inactivity over
an extended period of time.
