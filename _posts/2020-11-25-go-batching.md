---
layout: post
title:  "Batch Processing in Go"
date:   2020-11-25
---
# Batch Processing in Go

Batching is a common scenario developers come across. Basically, to split a large amount of work into smaller chunks for optimal processing.

Seems pretty simple, and it really is. Say, we have a long list of items we want to process in some way. A pre-defined number of them can be processed concurrently. I can see two different ways to do it in Go.

First, using plain old slices. This is something most developers have probably done at some point in their career.
Let's take this simple example:

```
func main() {
	data := make([]int, 0, 100)
	for n := 0; n < 100; n++ {
		data = append(data, n)
	}
	process(data)
}

func processBatch(list []int) {
	var wg sync.WaitGroup
	for _, i := range list {
		x := i
		wg.Add(1)
		go func() {
			defer wg.Done()
			// do more complex things here
			fmt.Println(x)
		}()
	}
	wg.Wait()
}

const batchSize = 10

func process(data []int) {
	for start, end := 0, 0; start <= len(data)-1; start = end {
		end = start + batchSize
		if end > len(data) {
			end = len(data)
		}
		batch := data[start:end]
		processBatch(batch)
	}
	fmt.Println("done processing all data")
}
```

The data to process is a plain list of integers. To keep things simple, we just want to print all of them, at most 10 concurrently. To achieve this, we loop over the list, divide it into chunks of `batchSize = 10` and process each batch serially. Short and sweet, and does what we want.

The second approach uses a buffered channel, similar to what's described [in this post on concurrency](https://medium.com/@deckarep/gos-extended-concurrency-semaphores-part-1-5eeabfa351ce). Let's look at the code first.

```
func main() {
	data := make([]int, 0, 100)
	for n := 0; n < 100; n++ {
		data = append(data, n)
	}
	batch(data)
}

const batchSize = 10

func batch(data []int) {
	ch := make(chan struct{}, batchSize)
	var wg sync.WaitGroup
	for _, i := range data {
		wg.Add(1)
		ch <- struct{}{}
		x := i
		go func() {
			defer wg.Done()
			// do more complex things here
			fmt.Println(x)
			<-ch
		}()
	}
	wg.Wait()
	fmt.Println("done processing all data")
}
```

This example uses a buffered channel of size 10. As each item is ready to be processed, it tries to *send* to the channel. Sends are blocked after 10 items. Once processed, it *reads* from the channel, thereby releasing from the buffer. Using a `struct{}{}` saves us some space, because whatever is sent to the channel never gets used.

As the author of [the post](https://medium.com/@deckarep/gos-extended-concurrency-semaphores-part-1-5eeabfa351ce) points out, here we're exploiting the properties of a buffered channel to limit concurrency. One might argue, this is not really batching, rather it's concurrent processing with a threshold. And I would totally agree. Regardless, it gets the job done and the code is tad simpler.

Is it any better than slices? Probably not. As for speed, I timed the execution of both programs, they ran pretty close. These examples are far too simple to see any significant difference in runtime. Channels in general are slower and more expensive than slices. Since there is no meaningful data being passed between the goroutines, it's probably a wasted effort. So why would I do it this way? Well, I like simple code. But that might not be enough of a reason. If the cost of serial processing of each batch outweighs the cost of using a channel, it might be worth a consideration!

*Note: Code snippets on this post are available [here](https://github.com/pnawalramka/pnawalramka.github.io-code-examples).*
