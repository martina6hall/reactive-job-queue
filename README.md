# Reactive Job Queue

A Reactive job queue backed by Redis.  This Job queue provides guarantees
(as much as Redis can provide) about the loss of job data. The job state
atomically changes in the database from `queued`, to `processing` to
`complete` so the data is always available in the database.

#### Get Started

`npm install reactive-job-queue`

Example: Producer

```JS
var JobQueue = require('reactive-job-queue');

var q = new JobQueue({ queuename: 'myjobqueue' });
q.send({"name": "test", "job": "data"}, function(error, result) {
	if (error) {
		// Should re-send or handle, if there was not an error.
		// the data was added to the queue and it is safe to continue
	}
});

```

Example: Consumer/Job processor

```JS
var JobQueue = require('reactive-job-queue');

var q = new JobQueue({ queuename: 'myjobqueue' });

q.registerProcessor(function(data) {
	yourProcessDataFunction(data, function(error, complete) {
		if (!error) {
			q.notifyJobComplete(data, function(error, data) {
				if (!error) {
					console.log("Processing complete!");
				}
			});
		}
	});
});
```

# API

### new ReactiveJobQueue(options)

Creates a new ReactiveJobQueue.

- `options` - (Object) Settings for this Job Queue, must be set as some
              members are mandatory
  - `queuename` - (String) The name of the queue to put/receive jobs to/from.
  - `port`      - (Integer|Optional) The redis port to connect to.
  - `host`      - (String) The port or hostname of the redis server.
  - `concurrency` - (Integer) The number of jobs to process at any time.

### send(job, callback)

Send a new Job to the queue.

- `job`      - (Object) the job to send to the queue as an object.
- `callback` - (Function) called once the data has been added to the queue, if
  the addition of the data failed the error argument is set. Takes two
  arguments: (error, result)

### registerProcessor(callback)

Register a function to process items on the queue as they arrive. Only one
function can be used to process items coming from the queue. Only the first
registered function will be used, everything else will be ignored.

- `callback` - (Function) the function to call with the job data from the queue.  This
  should accept a data argument: (data).  The data is the
  object sent from the client with an additional `__reactive_job_id` property
  as a 'uuid'.

### notifyJobComplete(job, callback)

Notify the job queue that a job has been processed successfully.  The job
object must be identical to the job received from the queue in the processor
function.

- `job`  - (Object) the job to move into the complete state
- `callback` - (Function) the callback to call when this operation completes.

# TODO

- Provide more generic job state transition.
- Provide mechanisms to recover jobs from each state.

# License

Samuel Giles - MIT Licensed
