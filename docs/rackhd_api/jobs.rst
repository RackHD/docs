Task Jobs
=============================

.. contents:: Table of Contents

A job is a javascript subclass with a run function that can be referenced
by a string. When a new task is created, and all of its validation and setup logic handled,
the remainder of its responsibility is to instantiate a new job class instance for
its specified job (passing down the options provided in the definition to the
job constructor) and run that job.

**Defining a Job**

To create a job, define a subclass of `Job.Base
<https://github.com/RackHD/on-tasks/blob/master/lib/jobs/base-job.js>`_
that has a method called
*_run* and calls *this._done()* somewhere, if the job is
not one that runs indefinitely.

.. code-block:: javascript

    // Setup injector
    module.exports = jobFactory;
    di.annotate(jobFactory, new di.Provide('Job.example'));
    di.annotate(jobFactory, new di.Inject('Job.Base');

    // Dependency context
    function jobFactory(BaseJob) {
        // Constructor
        function Job(options, context, taskId) {
            Job.super_.call(this, logger, options, context, taskId);
        }
        util.inherits(Job, BaseJob);

        // _run function called by base job
        Job.prototype._run = function _run() {
            var self = this;
            doWorkHere(args, function(err) {
                if (err) {
                    self._done(err);
                } else {
                    self._done();
                }
            });
        }

        return Job;
    }

Many jobs are event-based by nature, so the base job provides many helpers for
assigning callbacks to a myriad of AMQP events published by RackHD services, such
as DHCP requests from a specific mac address, HTTP downloads from a specific IP, template
rendering requests, etc.