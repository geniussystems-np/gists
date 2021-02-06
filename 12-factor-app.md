# 12 Factor App

## Why

- clean contract with underlying operating system - maximum portability
- continuos deployment - maximum agility
- scale up

## 1. Code Base

- one codebase tracked in revision control, many deploys
- If there are multiple codebases, it’s not an app – it’s a distributed system. Each component in a distributed system is an app, and each can individually comply with twelve-factor.
- Multiple apps sharing the same code is a violation of twelve-factor. The solution here is to factor shared code into libraries which can be included through the [dependency manager](https://12factor.net/dependencies).
- There is only one codebase per app
- Additionally, every developer has a copy of the app running in their local development environment, each of which also qualifies as a deploy

## 2. Dependencies

- Explicitly declare and isolate dependencies
- vendoring/bundling
- declares all dependencies, completely and exactly, via a *dependency declaration* manifest
- *dependency isolation*
- do not rely on the implicit existence of any system tools
- If the app needs to shell out to a system tool, that tool should be vendored into the app.

## 3. Config

- Store config in the environment
- **strict separation of config from code**
- Config varies substantially across deploys, code does not.
- can it be open sourced at any time? test
- **The twelve-factor app stores config in \*environment variables\***
- .env
- env vars are granular controls, each fully orthogonal to other env vars

## 4. Backing Services

- Treat backing services as attached resources
- ny service the app consumes over the network as part of its normal operation
- **The code for a twelve-factor app makes no distinction between local and third party services**
- Each distinct backing service is a *resource*

## 5. Build, Release, Run

- Strictly separate build and run stages
- build stage - dependenies -> make
- release -> combine with current config
- run -> runs the app in the execution environment

## 6. Processes

- Execute the app as one or more stateless processes
- **Twelve-factor processes are stateless and [share-nothing](http://en.wikipedia.org/wiki/Shared_nothing_architecture)**
- ticky sessions are a violation of twelve-factor and should never be used or relied upon.

## 7. Port binding

- Export services via port binding
- **The twelve-factor app is completely self-contained** and does not rely on runtime injection of a webserver into the execution environment to create a web-facing service. The web app **exports HTTP as a service by binding to a port**, and listening to requests coming in on that port
- using [dependency declaration](https://12factor.net/dependencies) to add a webserver library to the app, such as [Tornado](http://www.tornadoweb.org/) for Python, [Thin](http://code.macournoyer.com/thin/) for Ruby, or [Jetty](http://www.eclipse.org/jetty/) for Java and other JVM-based languages. This happens entirely in *user space*, that is, within the app’s code

## 8. Concurrency

- Scale out via the process model
- In the twelve-factor app, processes are a first class citizen
- Processes in the twelve-factor app take strong cues from [the unix process model for running service daemons](https://adam.herokuapp.com/past/2011/5/9/applying_the_unix_process_model_to_web_apps/)
- HTTP requests may be handled by a web process, and long-running background tasks handled by a worker process
-  [share-nothing, horizontally partitionable nature of twelve-factor app processes](https://12factor.net/processes)
- Twelve-factor app processes [should never daemonize](http://dustin.github.com/2010/02/28/running-processes.html) or write PID files.Instead, rely on the operating system’s process manager (such as [systemd](https://www.freedesktop.org/wiki/Software/systemd/), a distributed process manager on a cloud platform, or a tool like [Foreman](http://blog.daviddollar.org/2011/05/06/introducing-foreman.html) in development) to manage [output streams](https://12factor.net/logs), respond to crashed processes, and handle user-initiated restarts and shutdowns.

## 9. Disposability

- Maximize robustness with fast startup and graceful shutdown
- **The twelve-factor app’s [processes](https://12factor.net/processes) are \*disposable\*, meaning they can be started or stopped at a moment’s notice**
- **minimize startup time**
- Processes **shut down gracefully when they receive a [SIGTERM](http://en.wikipedia.org/wiki/SIGTERM)** signal from the process manager
- Processes should also be **robust against sudden death**
-  a twelve-factor app is architected to handle unexpected, non-graceful terminations. [Crash-only design](http://lwn.net/Articles/191059/) takes this concept to its [logical conclusion](http://docs.couchdb.org/en/latest/intro/overview.html).

## 10. Dev/Prod parity

- Keep development, staging, and production as similar as possible
- **The twelve-factor app is designed for [continuous deployment](http://avc.com/2011/02/continuous-deployment/) by keeping the gap between development and production small.**
- Make the time gap small: a developer may write code and have it deployed hours or even just minutes later.
- Make the personnel gap small: developers who wrote code are closely involved in deploying it and watching its behavior in production.
- Make the tools gap small: keep development and production as similar as possible.
- **The twelve-factor developer resists the urge to use different backing services between development and production**

## 11. Logs

- Treat logs as event streams
-  text format with one event per line (though backtraces from exceptions may span multiple lines
- **A twelve-factor app never concerns itself with routing or storage of its output stream.**
- each running process writes its event stream, unbuffered, to `stdout`

## 12. Admin processes

- Run admin/management tasks as one-off processes
- database migrations
- one-time scripts committed into the app’s repo
