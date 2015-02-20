# try
Idiomatic Go retry package

### Usage

Just call `try.Do` with the function you want to retry in the event of an error:

```
var value string
err := try.Do(func(attempt int) (error, bool) {
  var err error
  value, err = SomeFunction()
  return err, attempt < 5 // try 5 times
})
if err != nil {
  log.Fatalln("error:", err)
}
```

  * Call `try.Do` that returns an `error` and a `bool` indicating whether to retry or not
  * The `attempt` argument will start at 1 and count up
  * The function will be called repeatedly until error is `nil`, while `attempt < 5` (i.e. try 5 times)
  * `try.Do` returns the last error or `nil` if it was successful

#### Maximum retry limit

To avoid infinite loops, Try will ensure it only makes `try.MaxRetries` attempts. By default, this value is `10`, but you can change it:

```
try.MaxRetries = 20
```

To see if a `Do` operation failed due to reaching the limit, you can check the `error` with `try.IsMaxRetries(err)`.

#### Retrying panics

Try supports retrying in the event of a panic.

  * Use named return parameters
  * Set `retry` first
  * Defer the recovery code, and set `err` manually in the case of a panic
  * Use empty `return` statement at the end

```
var value string
err := try.Do(func(attempt int) (err error, retry bool) {
  retry = attempt < 5 // try 5 times
  defer func() {
    if r := recover(); r != nil {
      err = errors.New(fmt.Sprintf("panic: %v", r))
    }
  }()
  value, err = SomeFunction()
  return
})
if err != nil {
  log.Fatalln("error:", err)
}
```

#### Delay between retries

To introduce a delay between retries, just make a `time.Sleep` call before you return from the function if you are returning an error. If there is no error, it will not retry.

```
var value string
err := try.Do(func(attempt int) (error, bool) {
  var err error
  value, err = SomeFunction()
  if err != nil {
    time.Sleep(1 * time.Minute) // wait a minute
  }
  return err, attempt < 5
})
if err != nil {
  log.Fatalln("error:", err)
}
```