### Benchmark

An event, run by an institute or organization, wherein several researchers, students, and data scientists participate and evaluate their ML model in health. Each benchmark has a start time and generally an end time too.

### Benchmark host

A member of the host team who organizes a benchmark. In our system, it is a form of representing a user. This user can be in the organizing team of many benchmarks, and hence for each benchmark, its benchmark host will be different.

### Benchmark host team

A group of benchmark hosts organizes a benchmark. They are identified by a unique team name.

### Benchmark phase

A benchmark phase represents a distinct stage of the benchmark. Over a period of time, benchmark organizers have started to use the benchmark phase as a way to:

1.  Decide when to evaluate submissions on a subset of the test set or when to evaluate on the whole test set.
2.  Use different benchmark phases as different tracks of the same benchmark.

### Benchmark phase split

A benchmark phase split is a relation between a benchmarking phase and dataset splits for a benchmark with a many-to-many relation. This is used to set the privacy of submissions (public/private) to different dataset splits for different benchmark phases.

### Dataset

A dataset is the main entity on which an AI benchmark is based on. Participants are expected to make submissions corresponding to different splits of the corresponding dataset.

### Dataset split

A dataset is generally divided into different parts called dataset split. Generally, a dataset has three different splits:

- Training set
- Validation set
- Test set

### Docker container
A standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.

### Evaluation Script
The evaluation script is the core component of the benchmark, wherein the model is evaluated against a variety of metrics. The evaluation script is fully customizable and up to the host to define.

### Leaderboard
The leaderboard can be defined as a scoreboard listing the names of the teams along with their current scores. Currently, each benchmark has its own leaderboard.

### Phase

A benchmark can be divided into many phases (or benchmark phases). A benchmark phase can have the same or different start and end date than the benchmark start and end date.

### Participant

A member of the team competing against other teams for any particular benchmark. It is a form of representing a user. A user can participate in many benchmarks, hence for each benchmark, its participant entry will be different.

### Participant team

A group of one or more participants who are taking part in a benchmark. They are identified uniquely by a team name.

### Submission

A way of submitting your results to the platform, so that it can be evaluated and ranked amongst others. A submission can be public or private, depending on the benchmark.

### Submission worker

A python script which processes submission messages received from a queue. It does the heavy lifting task of receiving a submission, performing mandatory checks, and then evaluating the submission and updating its status in the database.

### Test annotation file

This is generally a file uploaded by a benchmark host and is associated with a benchmark phase. This file is used for ranking the submission made by a participant. An annotation file can be shared by more than one benchmark phase. In the codebase, this is present as a file field attached to benchmark phase model.

### Worker
A managed compute instances that perform the benchmark of the model.
