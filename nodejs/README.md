# Error handling TS decorator
```typescript
export function alwaysReturnComplete (target: any, propertyKey: string, descriptor: PropertyDescriptor): any {
  const originalMethod = descriptor.value
  const className: string = target.constructor.name.toString()
  descriptor.value =  (...args: any[]) => {
    if (originalMethod.constructor.name == 'AsyncFunction') {
      return originalMethod.apply(this, args).catch((error: any) => {
        console.error(`SQS Handler error caught in ${className} - ${propertyKey} \n\r`, error)
        return {
          status: ProcessingStatus.COMPLETE
        }
      })
    } else {
      try {
        return originalMethod.apply(this, args)
      } catch (error: any) {
        console.error(`SQS Handler error caught in ${className} - ${propertyKey} \n\r`, error)
        return {
          status: ProcessingStatus.COMPLETE
        }
      }
    }
    return descriptor
  }
}
```
## Explanation & Use Case
This function is a decorator which could be used like this in a test scenario
```typescript
    class test {
      @alwaysReturnComplete
      async testMethod () {
        throw new Error('Test Error')
        return { status: ProcessingStatus.SKIPPED }
      }
    }
```
The goal of the decorator is to override the return value of a function should the function return an error.
In short this use case came up because messages in an AWS SQS queue that don't return a valid response, 
get re-added to the SQS queue. For systems that are load sensitive, this is most definitely not wanted as the
continuous reprocessing of messages that make multiple calls to the load sensitive system, could overwhelm it resulting
in down time.

Hence this decorator was created after the fact for all SQS message processing functions, to ensure that in the event 
that an existing function doesn't implement proper error correction, it at least returns a "completed" status to the queue.

This decorator works for both Async and synchronous functions