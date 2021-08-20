## go-concurrent-job [![Go Reference][1]][2]
`go-concurrent-job` is an open-source tool that shows how to create a worker pool using channels and goroutines to provide maximum throughput.

This package consists of two abstractions `worker` and `dispatcher` both interact to get process a job distributed among several workers. You can run the example and see the difference.

avg time 10 sequential runs := 17.25387443 Seconds
avg time 10 concurrent runs := 1.494497171 Seconds

## Getting started
### Build and run it
Installation of `go-concurrent-job` is dead-simple, just download [the latest release] and build it.

### Example
A very simple example is here:

```go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"runtime"
	"time"

	"github.com/teocci/go-concurrent-job/dispatcher"
	"github.com/teocci/go-concurrent-job/worker"
)

const baseURL = "https://age-of-empires-2-api.herokuapp.com/api/v1/civilization/%d"

var terms = []int{
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
	1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4, 1, 2, 3, 4,
}

var c *http.Client

func main() {
	start := time.Now()
	poolNumber := runtime.NumCPU()
	fmt.Println(poolNumber)

	c = &http.Client{Timeout: time.Millisecond * 15000}
	dd := dispatcher.New(poolNumber).Start(CallApi)

	for i := range terms {
		dd.Submit(worker.Job{
			ID:        i,
			Name:      fmt.Sprintf("JobID::%d", i),
			CreatedAt: time.Now(),
			UpdatedAt: time.Now(),
		})
	}

	end := time.Now()
	log.Print(end.Sub(start).Seconds())
}

func CallApi(id int, job worker.Job) error {
	ur := fmt.Sprintf(baseURL, job.ID)
	req, err := http.NewRequest(http.MethodGet, ur, nil)
	if err != nil {
		//log.Printf("error creating a request for term %d :: error is %+v", num, err)
		return err
	}

	res, err := c.Do(req)
	if err != nil {
		//log.Printf("error querying for term %d :: error is %+v", num, err)
		return err
	}
	defer res.Body.Close()

	_, err = ioutil.ReadAll(res.Body)
	if err != nil {
		//log.Printf("error reading response body :: error is %+v", err)
		return err
	}

	log.Printf("%d  :: ok", id)
	return nil
}
```

[1]: https://pkg.go.dev/badge/github.com/teocci/go-concurrent-job.svg
[2]: https://pkg.go.dev/github.com/teocci/go-concurrent-job
[3]: https://github.com/teocci/go-concurrent-job/releases/tag/v1.0.0