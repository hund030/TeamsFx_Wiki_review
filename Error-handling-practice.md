# How to classify `UserError` and `SystemError`
UserError: errors caused by user's input/behavior, which can fixed by user's proper behavior. For example:
   - User's subscription has no permission on some resources
   - Invalid inputs
   - Concurrent operation error, please wait until the running task finishs or you can reload the window to cancel it.

SystemError: errors caused by system bugs or failures, which can not fixed by user. For example, 
   - Cannot read property ' [*] ' of undefined

For Errors thrown by third-party modules, it's better to analyze and classify the Error types: 
```
failed to exec dotnet script.,errorStack=Failed to install .NET Core SDK (v3.1). Install .NET Core SDK (v3.1) manually and restart Visual Studio Code
```
Frequent UserError means that user experience of the product should be improved to decrease the possibility of user using the toolkit in the wrong way.
Frequent SysteError means that the system should reduce bugs to improve the quality and increase the usability.


# How to define error

## Three basic method to define error

There are many ways to define customized errors in the project:
1. Define an error as a constant value
1. Define an error as a function that returns a `UserError` or `SystemError`
1. Define an customized error class
1. directly construct a `UserError` or `SystemError` in the place where error happens
    1. construct error (`UserError`/`SystemError`) with name, message,source,...
    1. construct error (`UserError`/`SystemError`) with existing Error object
    1. construct error (`UserError`/`SystemError`) with option (`UserErrorOptions`/`SystemErrorOptions`) object

**We don't suggest first method**, unless you don't care about the stack at all.
Because the error stack is constant and meaningless in such a case.

**We don't suggest the second method neither**, which add one more call stack on top of the stacks where the error really happens.

Here is a bad sample:
```
function MyError() {
  return new UserError("MyError", "my error message", "API");
}
console.log(MyError());
```
The error stack will contains the function `MyError`, which is not expected:

![image](https://user-images.githubusercontent.com/1658418/132477124-3e0904fb-2a06-485e-9e73-d61a5780e26c.png)

**We suggest the third method** to write your own error class that extends `UserError` or `SystemError`.

For example: 
```
class MyError extends UserError {
  constructor () {
    super(new.target.name, "my error message", "API")
  }
}
```
Then the stack will be clean: 

![image](https://user-images.githubusercontent.com/1658418/132477358-dfb459e9-513c-47c1-b33a-8d4696854fd6.png)

Another advantage of the third method is that it support `instanceof` operation on error type checking:
```
console.log(new MyError() instanceof MyError); // output `true`
```
Because in the constructor, we have passed `new.target.name` (which is actually `MyError`) as error name parameter into the `super` constructor.
Alternatively, if we don't want to define an error name different from the class name, we can just pass an empty string as the error name.
The following definition is equivalent to previous one:
```
class MyError extends UserError {
  constructor () {
    super("", "my error message", "API")
  }
}
```
The reason is the in constructor of `UserError`/`SystemError`, we have some priority checking on error name. Explicitly input name is the first priority, then the name of `Error` object is the second priority. If both are empty, we will use the constructor name as the default error name:
```
this.name = option.name || (option.error && option.error.name) || new.target.name;
```
In addition to method three, **We also suggest the fourth method**, because it is the most flexible. 

## Construct a UserError/SystemError

We have three override constructors for `UserError` or `SystemError`.  

### Constructor with `name`, `message` and `source` string
```
const error = new UserError(myName, myMessage, mySource, undefined, myHelpLink);
chai.assert.equal(error.name, myName);
chai.assert.equal(error.message, myMessage);
chai.assert.equal(error.source, mySource);
chai.assert.equal(error.helpLink, myHelpLink);
chai.assert.isDefined(error.timestamp);
chai.assert.isTrue(error instanceof UserError);
```

### Constructor with existing `Error` object
```
const innerError = new RangeError(myMessage);
const error = new UserError(innerError, mySource);
chai.assert.equal(error.name, "RangeError");
chai.assert.equal(error.message, myMessage);
chai.assert.equal(error.source, mySource);
chai.assert.isTrue(error instanceof UserError);
chai.assert.equal(error.innerError, innerError);
```
When constructing with existing Error object, the name of the input Error will have higher priority than the constructor name.

### Constructor with an option (`UserErrorOptions`/`SystemErrorOptions`) object
This approach provides most flexible way to construct a Error:
```
const error = new UserError({ error: new RangeError(myMessage), source: mySource, helpLink: myHelpLink });
chai.assert.equal(error.name, "RangeError");
chai.assert.equal(error.message, myMessage);
chai.assert.equal(error.source, mySource);
chai.assert.equal(error.helpLink, myHelpLink);
chai.assert.isTrue(error instanceof UserError);
```
You can input an empty option to build an error, the constructor will replace empty values with some default values:
```
const error = new UserError({});
chai.assert.equal(error.name, "UserError");
chai.assert.equal(error.message, "");
chai.assert.equal(error.source, "unknown");
chai.assert.isTrue(error instanceof UserError);
```


