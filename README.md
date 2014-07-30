# QUE?

Opinionated NSQ producer and consumer interfaces for Go. Dependencies are vendored for stability.

## Configuration

Environment variables must be set: `NSQD_HOST` and `NSQLOOKUPD_HOST`

## Producing (Writing to NSQ)

```go
  import "github.com/jcoene/que"

  // publish one message
  err = que.Publish("widgets", myStruct)

  // or publish many messages
  err = que.MultiPublish("widgets", []myStructSlice)
```

## Consuming (Reading from NSQ)

```go
  import (
    "fmt"

    "github.com/jcoene/queue"
  )

  // Widget is the data type that we're going to read from the queue
  type Widget struct {
    Id   string `json:"id"`
    Name string `json:"name"`
    Qty  int    `json:"qty"`
  }

  // Each topic needs a handler that implements to the Id and Perform methods
  type WidgetHandler struct {
    Widget *Widget
  }

  // The Id method returns a string used to represent the job (used for context).
  func (h *WidgetHandler) Id() string {
    return wp.Widget.Id
  }

  // The Perform method does the work. It returns error on failure, nil on success.
  func (wp *WidgetHandler) Perform() error {
    if wp.Qty < 0 {
      return fmt.Errorf("we don't have enough widgets")
    }

    return ShipWidget(wp.Widget)
  }

  // This does nothing, but does it very successfully.
  func ShipWidget(w *Widget) {
    return nil
  }

  // Each type needs a generator method that takes a payload and returns a stateful Handler and error (or nil).
  func newWidgetHandler(p *que.Payload) (que.Handler, error) {
    h := new(WidgetHandler)
    err := json.Unmarshal(p.Message.Body, &h.Widget)
    return h, err
  }

  func main() {
    manager := que.NewManager()

    widgetConsumer := que.NewConsumer("widgets", "myapp", 10, newWidgetHandler)
    widgetConsumer.Config.Set("some_nsq_config_setting", 5)
    manager.AddConsumer(widgetConsumer)

    manager.Run()
  }
```

