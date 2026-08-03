# You are on defence. Why do you need access to Sentry?

A recent incident changed my perspective on that.

Some context first.

This was one of my own applications. It was not highly critical from an availability perspective. If it went down, operations would survive. But it had access to databases containing PII, so a data leak could still create serious problems.

The stack looked like this:

* Python FastAPI backend
* Next.js frontend
* MySQL as the primary database
* Redis for frequently queried configuration values
* Redis Streams as a buffer for analytics events
* ClickHouse for analytics storage
* Elastic for security logging
* Prometheus for operational monitoring
* Sentry for application errors and stack traces

The application ran on ephemeral containers. If one failed, another would spin up.

It also received roughly five to ten version updates on an active day. Some were code changes, while others were update part of the content process pipeline (think of this as a json file with rules on how to process data).

That detail matters.

An attacker found a broken access control issue in the application.

I started with the normal checks in Elastic.

Too many requests?

Strange traffic patterns?

Suspicious alerts?

Anything obviously out of place?

There were a couple of 405 Method Not Allowed responses involving POST requests.

Not enough for me to immediately think, “What is happening here?”

The activity was not noisy. The attacker was not hammering every endpoint or generating thousands of requests. They made only a small number of requests, enough to test and exploit the issue without attracting much attention.

At least they were not exhibiting complete noob behaviour.

So the battle of the noobs never started.

The timing made it even harder to notice. The requests appeared shortly after a version update.

Because I had built the application myself, I had a bias while reviewing the logs. The developer in me treated the 405 responses as a possible side effect of the update.

One of the modules also had an authorization design where even internal requests passed through a permission layer. Permission-related failures were therefore not completely unusual during development or content updates.

I suspect the frequency of updates made me pay less attention than I would have if I were looking at the application purely from a security perspective.

The request was also a POST request, so the useful request body was not available in the Elastic logs.

From Elastic alone, I had a few 405 responses, low request volume, and activity that appeared after a deployment. Nothing screamed security incident.

What changed the investigation was Sentry.

The failed authorization flow triggered an application exception, and Sentry captured the complete stack trace.

That stack trace was gold.

It showed the full execution chain:

* Which endpoint was called
* Which internal functions were triggered
* Which authorization checks were performed
* Where the request failed
* How the application reached that state

From there, I correlated the timestamps with Elastic, narrowed the activity down to a specific user account, and reconstructed the full chain of events.

The lesson was not that Sentry is better than Elastic.

The lesson was understanding what information you can get, where you can get it, and how to connect the evidence.

Elastic showed the surrounding requests.

Sentry showed what happened inside the application.

Prometheus showed whether the system itself was unhealthy.

The application architecture explained why the failure mattered.

The deployment history explained why the event initially looked normal.

None of those systems had the complete answer by themselves.

The answer came from understanding the architecture, the development process, and the visibility provided by each tool.

Over the past few months, I have spent much more time in development mode. Because of that, some errors that might interest a security analyst no longer immediately stand out to me. I see similar failures regularly while building and updating systems.

But development has also changed how I investigate security incidents.

I no longer look only at alerts.

I think about how requests move through the application, where authorization happens, what changed in the latest release, what each tool can see, and what evidence may exist outside the SIEM.

I have also started building threat models at the module level during development (this is part of the agents.md file).

Those threat models are then used to create security, functional, and QA test cases before code reaches production.

The current problem is that every commit to main now has to survive more than 500 tests.

It is a lot.

But it is still cheaper than learning about broken access control from an attacker.
