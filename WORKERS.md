# workers

## Persistent data model
Every worker has a fixed and permanent `id` which both represents a unique identification method and its sorting ID. It is changeable, however only to a value that is not used yet. If you want to swap the IDs of two workers, you first of all need to change the ID of one of them to a free one.
It also has a randomly generated and hashed stored `token` which is used to establish a connection in the name of that worker. This token cannot be copied after the initial creation, so you would have to regenerate it if needed.
There will be a list of whitelisted IPs too, however it is optional. If at least one IP is added to that list, only incoming connections from one of these are allowed. This does **not** affect the requirement of the secret token.

## Connections
A worker is connected to the server by the general WebSocket-based communication interface. The initial packet says it is a worker and contains the ID + the secret token to allow proper access control validation.
If the given values are valid and the requestor is eligible to establish a connection, the server hands the connection to an internal connection handler dedicated to worker connections.
The connection is constantly validated by a heartbeat requirement.

## Pipeline job execution process
When a pipeline job execution is triggered on the servers side, it hands this request on to an internal **job queue**. Once it is the first element in this queue and a worker gets available, this request is handed on to the worker itself, along with every single pre-defined data:

- Base Docker image
- Optional required additional packages (summarized of all stages)
- File replacements
- All secrets of the pipeline
- Detailed instructions about every single job stage
    - Commands to execute
    - Artifacts to archive
    - Whether or not log filtering should be applied

Once received, the worker acknowledges the request retrieval and starts working on it in multiple stages:

1. The **preparation** stage is used to set up the required execution environment:
    - A Docker container with the given base image is started
    - The defined required packages are being injected
    - File replacements get injected
    - The source code gets pulled
2. The **running** stage basically consist of all pipeline stages. These get executed in the defined row and commands get edited beforehand to include secrets. After every successful stage execution all defined artifacts get pushed into a local job-wide artifact storage. If enabled, all occurances of secrets in the log output get removed before handing out these. After running all stages the worker marks the job as complete and returns all relevant resulting data like artifacts.
3. The **cleanup** stage runs after the execution itself. It stops the Docker container, removes it and the used image and generally tydies up the worker.

**Note:** These stages are **worker stages**. The **job stages** are split up into the first two parts of it: **preparation** and **running** (the second split up into the single pipeline stages). During these two parts of the worker stage a job is currently being worked on and so the worker constantly pushes updates (logs and job status) to the server. After the running stage the job is being marked as done, but the worker itself is not available yet as it has to run through the cleanup stage.