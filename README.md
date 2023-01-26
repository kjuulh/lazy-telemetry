# Lazy telemetry

is a small tool/server, for storing and collecting stats on various tools you
use. It will track how many times a cli function is invoked, crashes occuring
and so on. It will store these locally, until at some point it gets an internet
connection, and then it will upload them to the server.

Everything is anonymous, and only used by a very tiny process to collect
telemetry stats.

There are two different modes. telemetry tracking, and error probes.

## telemetry

telemetry tracking:

```go
func main() {
  if err := telemetry.Increment("some-app"); err != nil {
    // returns err, but can be ignored, and will never panic
  }

  // ... Rest of code
}
```

Error probe:

```go
func main() {
  action := func() error {
    cli.Execute()
  }

  if err := telemetry.Monitor("some-app", action); err != nil {
    // Will also count
    panic(err)
  }
}
```

### Install the upload

This is not exactly required, but will greatly help getting all the stats
uploaded consistently. Because the connection will be unstable, each statement
requires a commit from the server. This slows down the upload, but greatly helps
consistency.

in your .zshrc or cronjob:

```sh
lazy-telemetry flush # will always return 0, but will log to ~/.local/state/telemetry/telemetry-cli.log
```

## Architecture

The telemetry is stored in .local/state/telemetry.

The output will look like this:

```json
{ "type": "increment", "timestamp": "some-time", "app": "some-app", "args": "app do something" }
{ "type": "telemetry", "timestamp": "some-time", "app": "some-app", "error": "error: something\nstack trace...", "args": "app do something" }
```
