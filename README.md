# lci concept

## 1.) What is `lci`?
`lci`, initially chosen as a short form of `LightCI`, is an attempt of me ([lus](https://github.com/lus)) to dig into the concepts and processes a CI/CD solution consists of. Generelly the topic of automation of software development really interests me so I decided to start this project to learn a bit more about it.

## 2.) Basic structure of an `lci` instance
An `lci` instance will consist of three essential parts:
- `server`:
    - The server is probably the most important part of the whole solution. First of all it **keeps track of all workers**. You can connect them to the server and it then assigns jobs to them (depending on their current status) or queues them if needed. It also forwards all needed credential data and receives and processes the job results in the end.
    - A second very important part is **general pipeline and build tracking**. Obviously, workers aren't responsible for the pipelines themselves. The server saves the pipelines, receives and processes build requests and generally updates and keeps track of the current state.
    - **Secret and credential management** is also a task handled by the server. I currently think about doing that with a Vault integration, but that would just be another layer to add so I'm quite unsure about that.
    - To provide a **secure and efficient access control** I have set myself the goal of integrating a **simple** but also **powerful role-based access control structure**. You create users, assign them to roles with several permissions and you are good to go.
    - Last but not least the workers and other services also need a unify interface to communicate with the central instance server. This is done by exposing a **fully-featured and access-controlled WebSocket-based communication interface**.
- `worker`:
    - The worker mainly **executes assigned jobs**. These jobs can consist of multiple **stages**, for example **building**, **testing** and **deployment**. The server requests a job execution and provides all relevant data related to it. After then it constantly gets notified about execution updates.
- `web`
    - I think **a sleak and modern webinterface** should be part of a CI/CD solution, so that is a service I definitely aim at.

## 3.) BUT THERE IS JENKINS, AND CIRCLECI, AND TRAVISCI, AND GITHUB ACTIONS, AND GITLAB CI AND...
shut up

## 4.) Cool, but I would do ... differently. Aaaand I would also integrate ...
Nice, I'm happy you're interested in this project. If you want, you can create and issue with your suggestions as I am pretty new to this topic as well.