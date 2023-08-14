# golock

## A lock implement with context and multiple keys supported

### 1. Get the module
    ```shell
    go get github.com/imstevez/golock
    ```

### 2. CtxRWMutex
    ```go
    package main

    import (
        "context"
        "github.com/imstevez/golock"
    )

    type Counter struct {
        count int
        mu  golock.CtxRWMutex
    }

    func (c *Counter)Add(ctx context.Context, delta int) (err error) {
        err = c.mu.Lock(ctx)
        if err != nil {
            return
        }

        defer c.mu.Unlock()

        c.count += delta

        return
    }

    func (c *Counter)Catch(ctx context.Contex, catch(int)) (err error) {
        err = c.mu.RLock(ctx)
        if err != nil {
            return
        }

        defer c.mu.RUnlock()

        catch(c.count)

        return
    }
    ```

### 3. MultiCtxRWMutex
    ```go
    //...
    
    type Counters struct{
        counts []int
        mus *golock.MultiCtxRWMutex
    }

    func NewCounters(n int) *Counters {
        return &Counters{
            counts: make([]int, n),
            mus:    golock.NewDefaultMultiCtxRWMutex(),
        }
    }
        

    func (cs *Counters) Add(ctx context.Context, idx int, delta int) (err error) {
        if idx < 0 || idx >= len(cs.counts) {
            err = errors.New("invalid idx")
            return
        }

        key := idx

        err = cs.mus.Lock(ctx, key)
        if err != nil {
            return
        }

        defer cs.mus.Unlock(key)

        cs.counts[idx] += delta

        return
    }

    func (cs *Counters) Catch(ctx context.Context, idx int, catch func(int)) (err error) {
        if idx < 0 || idx >= len(cs.counts) {
            err = errors.New("invalid idx")
            return
        }

        key := idx

        err = cs.mus.RLock(ctx, key)
        if err != nil {
            return
        }

        defer cs.mus.RUnlock(key)

        catch(cs.counts[idx])

        return
    }
    
    ```


