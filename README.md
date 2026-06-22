<div align="center">

# QBWC Rails Interface

A Rails interface for Intuit's QuickBooks Web Connector, enabling Rails applications to exchange data with QuickBooks Desktop.

</div>

## 📌 Overview

QBWC helps a Rails application communicate with QuickBooks Desktop through Intuit's QuickBooks Web Connector. In this workflow, QuickBooks Web Connector acts as the HTTP client, while the Rails application acts as the HTTP server.

The project provides Rails setup helpers, database-backed jobs, worker classes, session handling, authentication configuration, and error-handling behavior for QuickBooks Web Connector integrations.

## ✨ Features

- Rails integration for QuickBooks Web Connector
- Generator-based setup for configuration and migrations
- Database-persisted jobs for QuickBooks tasks
- Worker classes for defining QuickBooks requests and handling responses
- Optional authentication for multiple users or company files
- Support for passing serializable data into jobs
- Session initialization and session completion hooks
- Configurable behavior when QuickBooks returns an error

## 🧰 Technology

| Area | Details |
| --- | --- |
| Framework | Ruby on Rails |
| Package type | Ruby gem |
| External software | Intuit QuickBooks Web Connector |
| Target system | QuickBooks Desktop |

## 🚀 Installation

Install the gem:

`gem install qbwc`

Or add it to a Rails application's Gemfile:

`gem "qbwc"`

Then install dependencies:

`bundle install`

Run the Rails generator:

`rails generate qbwc:install`

Run the database migrations:

`rake db:migrate`

After setup, review `config/initializers/qbwc.rb` and restart the Rails application.

QuickBooks requires HTTPS when connecting to remote machines. A tunneling service such as ngrok may be useful during development.

## 🔐 Authentication

QBWC can be configured with an authenticator for multiple users or multiple QuickBooks company files. The authenticator is a `Proc` configured in `config/initializers/qbwc.rb`.

It receives a username and password, then returns the path to the QuickBooks company file the user may access. Returning `nil` denies access.

This is useful when different users should connect to different QuickBooks company files, or when separate credentials are needed for different QuickBooks Web Connector configurations.

## 🧾 QuickBooks Web Connector Setup

Install QuickBooks Web Connector on the machine where QuickBooks Desktop is installed.

For a single-user, single-company setup, visit `/qbwc/qwc` on the Rails application's HTTPS domain and download the QWC file provided by the application. In QuickBooks Web Connector, choose “Add an application” and select that QWC file.

For multiple users or company files, the QWC file may need manual edits:

- Change `OwnerID` when multiple people connect to the same company file.
- Change `UserName` when using separate logins.
- Change `AppName` and `FileID` when connecting to multiple company files.

The generated `QbwcController` can customize `FileID` and `AppName` by overriding the related methods.

## ⚙️ Jobs

Jobs describe work that QuickBooks Web Connector should perform during an update session.

A job includes:

- A unique job name
- Whether the job is enabled
- The QuickBooks company file path
- A worker class
- Optional request data
- Optional serializable data passed to the worker

Jobs are persisted in the database and remain active until disabled or deleted. A worker may delete or disable its job after completion if the work should only run once.

## 🧑‍💻 Workers

A worker descends from `QBWC::Worker` and can define three main methods:

- `requests(job, session, data)` returns the QuickBooks request or requests.
- `should_run?(job, session, data)` controls whether the job should run.
- `handle_response(response, session, job, request, data)` handles the QuickBooks response.

Requests use underscored, lowercased versions of qbXML tags, such as `customer_query_rq` instead of `CustomerQueryRq`.

The Intuit Onscreen Reference for Intuit Software Development Kits can be used to find qbXML request and response formats.

## 🔄 Sessions

QBWC supports optional session initialization. Initialization code can run once when a QuickBooks Web Connector session starts, before queued jobs are executed.

A session initializer can be configured in `config/initializers/qbwc.rb` or set in application code with `QBWC.set_session_initializer`.

QBWC can also run a block after a session completes successfully by assigning `QBWC.session_complete_success`.

## ⚠️ Error Handling

By default, when QuickBooks returns an error response, the worker's response handler is invoked, but remaining requests in the current job and later jobs are not processed during that session.

The job remains persisted and may run again during the next QuickBooks Web Connector session.

To continue processing after an error, configure `on_error` as `:continue` in `config/initializers/qbwc.rb`.

## ✅ Typical Use Cases

- Connecting a Rails application to QuickBooks Desktop
- Querying customers or other QuickBooks records through qbXML
- Running repeatable QuickBooks Web Connector jobs
- Supporting multiple QuickBooks company files
- Handling QuickBooks responses inside Rails worker classes
- Running initialization logic at the start of a Web Connector session

## ❓ FAQ

### What is QBWC used for?

QBWC is used to connect a Rails application with QuickBooks Desktop through Intuit's QuickBooks Web Connector.

### Does QuickBooks Web Connector call the Rails app?

Yes. QuickBooks Web Connector acts as the HTTP client and sends requests to the Rails application, which acts as the server.

### Are jobs stored permanently?

Jobs are persisted in the database and remain active until disabled or deleted.

### Can different users access different QuickBooks company files?

Yes. QBWC supports an authenticator that can map usernames and passwords to specific QuickBooks company file paths.

### Does QBWC require HTTPS?

QuickBooks requires HTTPS connections when connecting to remote machines.

### How are QuickBooks requests defined?

Requests can be defined inside a `QBWC::Worker` or passed directly when adding a job.

## 🤝 Contributing

Contributions should be based on the latest master branch. Before opening a change, check whether the feature or bug has already been addressed, create a feature or bugfix branch, add tests where appropriate, and run the test suite with:

`rake test`
