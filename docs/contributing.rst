Contributing to RackHD
=============================

.. contents:: Table of Contents
   :depth: 2

We certainly welcome and encourage contributions in the form of issues and pull requests, but please read the
guidelines in this document before you get involved.

Since our project is relatively new, we don't yet have many hard and fast rules. As the project grows and more
people get involved, we will solidify and extend our guidelines as needed.


Communicating with Other Users
------------------------------

We maintain a mailing list at https://groups.google.com/d/forum/rackhd. You can visit the group through the web page or subscribe directly by sending email to rackhd+subscribe@googlegroups.com.

We also have a #RackHD slack channel at https://codecommunity.slack.com/messages/rackhd/. You can receive an invite by requesting one at http://community.codedellemc.com/.


Submitting Contributions
-----------------------------


To submit coding additions or changes for a repository, fork the repository and clone it locally. Then use a unique branch to make commits and send pull requests.

Keep your pull requests limited to a single issue. Make sure that the description of the pull request is clear and complete.

Run your changes against existing tests or create new ones if needed. Keep tests as simple as possible.  At a minimum, make sure your changes don’t break the existing project.
For more information about contributing changes to RacKHD, please see :doc:`devguide/contributing`

After receiving the pull request, our core committers will give you feedback on your work and may request that you make further changes and resubmit the request. The core committers will handle all merges.

If you have questions about the disposition of a request, feel free to email one of our core committers.

Core Committer Team
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Michael.Hepfer@dell.com
* Andrew.Hou@dell.com
* Andre.Keedy@dell.com
* James.King@dell.com
* Lyne.Lin@dell.com
* Rahman.Muhammad@dell.com
* Jeanne.Ohren@dell.com
* Geoffrey.Reid@dell.com
* Stuart.Stanley@dell.com
* James.Turnquist@dell.com



Please direct general conversation about how to use RackHD or discussion about improvements and features to our mailing list at rackhd@googlegroups.com


Issues and Bugs
-----------------------------

Please use https://rackhd.atlassian.net/secure/RapidBoard.jspa?rapidView=5 to raise issues, ask questions, and report bugs.

Search existing issues to ensure that you do report a topic that has already been covered. If you have new information to share about an existing issue, add your information to the existing discussion.

When reporting problems, include the following information:

* Problem Description
* Steps to Reproduce
* Actual Results
* Expected Results
* Additional Information


Security Issues
-----------------------------

If you discover a security issue, please report it in an email to rackhd@emc.com. Do not use the Issues section to describe a security issue.


Understanding the Repositories
------------------------------

The https://github.com/rackhd/RackHD repository acts as a single source location to help you get or build all the pieces to learn about, take advantage of, and contribute to RackHD.

A thorough understanding of the individual repositories is essential for contributing to the project. The repositories are described in our documentation
at :doc:`devguide/repositories`.


Submitting Design Proposals
-----------------------------

Significant feature and design proposals are expected to be proposed on the mailing list (rackhd@googlegroups.com, or at groups.google.com/forum/#!forum/rackhd)
for discussion. The Core Committer team reviews the proposals to make sure architectural details are aligned, with a floating agenda updated on the
RackHD Confluence page at https://rackhd.atlassian.net/wiki/spaces/RAC1/pages/9437198/Core+Commiter+Weekly+Interlock (formerly github wiki at https://github.com/RackHD/RackHD/wiki/Core-Committer-Meeting). The meeting notes are posted to the google groups mailing list.

Work by dedicated teams is scheduled within a broader `RackHD Roadmap`_. External contributions are absolutely welcome outside of planning exposed in the
roadmap.

.. _RackHD Roadmap: https://github.com/RackHD/RackHD/wiki/roadmap




Coding Guidelines
-----------------------------

Use the same coding style as the rest of the codebase. In general, write clean code and supply meaningful and comprehensive code comments. For more
detailed information about how we've set up our code, please see our :doc:`devguide/index`.


Contributing to the Documentation
---------------------------------

To contribute to our documentation, clone the `RackHD/docs`_ repository and submit commits and pull requests as is done for the other repositories.
When we merge your pull requests, your changes are automatically published to our documentation site at http://rackhd.readthedocs.org/en/latest/.

.. _RackHD/docs: https://github.com/RackHD/docs



Community Guidelines
-----------------------------

This project adheres to the `Open Code of Conduct`_. By participating, you are expected to honor this code.
Our community generally follows `Apache voting guidelines`_ and utilizes `lazy consensus`_ for logistical efforts.

.. _Open Code of Conduct: http://todogroup.org/opencodeofconduct/#RackHD/rackhd@emc.com
.. _Apache voting guidelines: http://www.apache.org/foundation/voting.html
.. _lazy consensus: http://en.osswiki.info/concepts/lazy_consensus
