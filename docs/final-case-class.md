Why should we make case class final?

```scala
case class MyClass(param: String)
class AnotherClass(param: String, anotherParam: Int) extends MyClass("anotherClass")
```
Then you get weird behaviour with equals and toString. Things like this can occur:

```scala
println(new AnotherClass("blah", 1)  ==  new AnotherClass("blah", 2))

res> true
```