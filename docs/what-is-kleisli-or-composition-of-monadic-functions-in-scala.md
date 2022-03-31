Hi All,

Hope you are doing great!

In this blog post, I am going to explain what is Kleisli or composition of monadic functions and how to use it. Before understanding Kleisli, let&#8217;s learn how to define functions and composition of the functions in Scala.

<pre><code class="scala">val getDataFromDb: Int =&gt; Int =(id: Int) =&gt; 10 //Assuming it will return Int value from DB
val processNumber: Int =&gt; Int       = (v: Int) =&gt; v * 2
val writeDataToDB: Int =&gt; Boolean = (v: Int) =&gt; true // Assuming succssfully db write will return Boolean value
</code></pre>

We have learnt how to define functions in Scala. Here you might be thinking why we have defined functions instead of methods? because in Scala, Functions and Methods both are not the same thing if I am not wrong. Methods are not values and you can not compose them without eta expansion. If you want, you can learn more about <a href="https://medium.com/@sinisalouc/on-method-invocations-or-what-exactly-is-eta-expansion-1019b37e010c" target="_blank" rel="noopener">why/what eta expansion </a>and <a href="https://tpolecat.github.io/2014/06/09/methods-functions.html" target="_blank" rel="noopener">why methods are not functions</a>.

Now I am hoping, you have understood functions. Nowadays, People are crazy to talk about Functional programming and there is a lot of stuff over the internet to explain what is functional programming and why it needs now but there is a brief definition of functional programming by <span class="username u-dir" dir="ltr"><a class="ProfileHeaderCard-screennameLink u-linkComplex js-nav" href="https://twitter.com/jdegoes" target="_blank" rel="noopener">@<b class="u-linkComplex-target">jdegoes</b></a> </span>

<img loading="lazy" class="alignnone wp-image-192" src="http://www.learnscala.co/wp-content/uploads/2018/10/IMG_6609-273x300.jpg" alt="" width="302" height="332" srcset="http://www.learnscala.co/wp-content/uploads/2018/10/IMG_6609-273x300.jpg 273w, http://www.learnscala.co/wp-content/uploads/2018/10/IMG_6609.jpg 747w" sizes="(max-width: 302px) 100vw, 302px" /> 

**&#8220;The rest is just composition you can learn over time&#8221;** this is what the topic of this blog post.

Let&#8217;s take an example:

<pre><code class="scala">scala&gt; val f: String =&gt; String = (s: String) =&gt; "f(" + s + ")"
f: String =&gt; String = $$Lambda$1078/554280593@e521067

scala&gt; val g: String =&gt; String = (s: String) =&gt; "g(" + s + ")"
g: String =&gt; String = $$Lambda$1079/753170002@65d90b7f

scala&gt; val fComposeG = f compose g // It is similar to call f(g("Hello,World"))
fComposeG: String =&gt; String = scala.Function1$$Lambda$1080/820069375@452ec287

scala&gt; fComposeG("Hello,World")
res1: String = f(g(Hello,World))

scala&gt; val fAndThenG = f  andThen g  // It is similar to g(f("Hello,World"))
fAndThenG: String =&gt; String = scala.Function1$$Lambda$1065/1796415927@40c6d1ef

scala&gt; fAndThenG("Hello,World")
res2: String = g(f(Hello,World))
</code></pre>

All functions that have single input are syntactic sugar of <a href="https://www.scala-lang.org/api/2.12.7/scala/Function1.html" target="_blank" rel="noopener">Function1[-T1, +R]</a> and it has two functions

<pre><code class="scala">def andThen[A](g: (R) ⇒ A): (T1) ⇒ A
Composes two instances of Function1 in a new Function1, with this function applied first.

def compose[A](g: (A) ⇒ T1): (A) ⇒ R
Composes two instances of Function1 in a new Function1, with this function applied last.
</code></pre>

Now I am hoping, you have understood how to compose functions. now let&#8217;s work on the first example.

<pre><code class="scala">scala&gt; val result = getDataFromDb andThen processNumber andThen writeDataToDB
result: Int =&gt; Boolean = scala.Function1$$Lambda$1065/1796415927@25291901

scala&gt; result(12)
res3: Boolean = true
</code></pre>

Sometimes you want your output in some context to delay your processing or want to run the program in effect. If you want to learn about effects here is awesome talk about <a href="https://www.youtube.com/watch?v=GZRL5Z40w60" target="_blank" rel="noopener">Functional Programming with Effects</a>

Let&#8217;s define functions to return monadic value i.e return value in a context, for instance, an **Option, Either, Try, Future, IO, Reader, Writer** etc:

<pre><code class="scala">//Below are the Monadic Functions

val getDataFromDbOpt: Int =&gt; Option[Int] = (id: Int) =&gt; Some(10) 
val processNumberOpt: Int =&gt; Option[Int]       = (v: Int) =&gt; Some(v * 2)
val writeDataToDBOpt: Int =&gt; Option[Boolean] = (v: Int) =&gt; Some(true)

val result = getDataFromDbOpt andThen processNumberOpt andThen writeDataToDBOpt
result(12)

//It will give you below compilation error
error: type mismatch;
 found   : scala.this.Function1[scala.this.Int,scala.this.Option[scala.this.Int]]
 required: scala.this.Function1[scala.this.Option[scala.this.Int],?]
  val result = getDataFromDbOpt andThen processNumberOpt andThen writeDataToDBOpt
</code></pre>

Ops, this is not what we are expecting. let&#8217;s fix this an error.

<pre><code class="scala">final case class KleisliForOption[A, B](run: A =&gt; Option[B]) {
  def andThen[C](k: KleisliForOption[B, C]): KleisliForOption[A, C] =
    KleisliForOption { a =&gt; run(a).flatMap(b =&gt; k.run(b)) }
}

val getDataFromDbOpt: Int =&gt; Option[Int]     = (id: Int) =&gt; Some(10)
val processNumberOpt: Int =&gt; Option[Int]     = (v: Int) =&gt; Some(v * 2)
val writeDataToDBOpt: Int =&gt; Option[Boolean] = (v: Int) =&gt; Some(true) 

val result = KleisliForOption(getDataFromDbOpt) andThen KleisliForOption(processNumberOpt) andThen KleisliForOption(writeDataToDBOpt)
result.run(12) //Output Some(true)</code></pre>

Now you can see, we are able to compose functions that have returned monadic value **Option**. what if, your functions return monadic value **Either**  
For example,

<pre><code class="scala">val getDataFromDbEither: Int =&gt; Either[String,Int]     = (id: Int) =&gt; Right(10)
val processNumberEither: Int =&gt; Either[String,Int]     = (v: Int) =&gt; Right(v * 2)
val writeDataToDBEither: Int =&gt; Either[String,Boolean] = (v: Int) =&gt; Right(true)

val result = KleisliForOption(getDataFromDbEither) andThen KleisliForOption(processNumberEither) andThen KleisliForOption(
  writeDataToDBEither)
result.run(12) 
//It fails with below compilation an error
Error:(12, 112) type mismatch;
 found   : Int =&gt; Either[String,Int]
 required: ? =&gt; Option[?]
val result = KleisliForOption(getDataFromDbEither) andThen KleisliForOption(processNumberEither) andThen KleisliForOption(writeDataToDBEither) </code></pre>

Let&#8217;s fix this error as well,

<pre><code class="scala">final case class KleisliForEither[A, B,E](run: A =&gt; Either[E,B]) {
  def andThen[C](k: KleisliForEither[B, C,E]): KleisliForEither[A, C,E] =
    KleisliForEither { a =&gt; run(a).flatMap(b =&gt; k.run(b))
    }
}

val getDataFromDbEither: Int =&gt; Either[String,Int]     = (id: Int) =&gt; Right(10)
val processNumberEither: Int =&gt; Either[String,Int]     = (v: Int) =&gt; Right(v * 2)
val writeDataToDBEither: Int =&gt; Either[String,Boolean] = (v: Int) =&gt; Right(true)

val result = KleisliForEither(getDataFromDbEither) andThen KleisliForEither(processNumberEither) andThen KleisliForEither(
  writeDataToDBEither)
result.run(12) //Output Either[String,Boolean] = Right(true)</code></pre>

What if your functions return monadic value like Option, Either, Try, <a href="https://www.scala-lang.org/api/2.12.7/scala/concurrent/Future.html" target="_blank" rel="noopener">Future</a>, <a href="https://typelevel.org/cats-effect/datatypes/io.html" target="_blank" rel="noopener">IO</a>, Reader, Writer etc then you have to define a similar structure for every monadic value as I did. let&#8217;s not define our structure because the <a href="https://typelevel.org/cats/datatypes/kleisli.html" target="_blank" rel="noopener">same structure</a> is already defined with <a href="https://typelevel.org/cats/" target="_blank" rel="noopener">cats library</a> and it is much powerful then what I have defined.

**Kleisli Type Signature:**

The Kleisli type is a wrapper around, A=>F[B]

<pre><code class="scala">final case class Kleisli[F[_], A, B](run: A =&gt; F[B])</code></pre>

Where F is some context that is a Monad, A is an Input and B is a output.

<pre><code class="scala"> import cats.Id
  import cats.effect.IO
  import cats.data.Kleisli
  import cats.implicits._

   //For Option
  val getDataFromDbOpt: Int =&gt; Option[Int]     = (id: Int) =&gt; Some(10)
  val processNumberOpt: Int =&gt; Option[Int]     = (v: Int) =&gt; Some(v * 2)
  val writeDataToDBOpt: Int =&gt; Option[Boolean] = (v: Int) =&gt; Some(true)

  val resultOpt = Kleisli(getDataFromDbOpt) andThen Kleisli(processNumberOpt) andThen Kleisli(
    writeDataToDBOpt)
  resultOpt.run(12) //Option[Boolean]

  //For Id monad
  val getDataFromDbFun: Kleisli[Id, Int, Int]     = Kleisli.apply(id =&gt; 10: Id[Int]) // I did because compiler is not infering type
  val processNumberFun: Kleisli[Id, Int, Int]     = Kleisli.apply(v =&gt; (v * 2): Id[Int])
  val writeDataToDBFun: Kleisli[Id, Int, Boolean] = Kleisli.apply(v =&gt; true: Id[Boolean])

  val resultFunc = getDataFromDbFun andThen processNumberFun andThen writeDataToDBFun
  resultFunc.run(12) //Id[Boolean] which is Boolean

 //For Either
  type ET[A] = Either[String, A]
  val getDataFromDbEither: Kleisli[ET, Int, Int] =
    Kleisli[ET, Int, Int](id =&gt; Right(10))

  val processNumberEither: Kleisli[ET, Int, Int]     = Kleisli[ET, Int, Int](v =&gt; Right(v * 2))
  val writeDataToDBEither: Kleisli[ET, Int, Boolean] = Kleisli[ET, Int, Boolean](v =&gt; Right(true))

  val result = getDataFromDbEither andThen processNumberEither andThen writeDataToDBEither
  result.run(12) //Either[String,Boolean]

  import scala.concurrent.Future
  import scala.concurrent.ExecutionContext.Implicits.global

  //For Future
  val getDataFromDbFuture: Kleisli[Future, Int, Int] = Kleisli.apply(_ =&gt; Future.successful(10))
  val processNumberFuture: Kleisli[Future, Int, Int] = Kleisli.apply(v =&gt; Future.successful(v * 2))
  val writeDataToDBFuture: Kleisli[Future, Int, Boolean] =
    Kleisli.apply(_ =&gt; Future.successful(true))

  val resultFuture = getDataFromDbFuture andThen processNumberFuture andThen writeDataToDBFuture
  resultFuture.run(12) //Future[Boolean]

  //For IO
  val getDataFromDbIO: Kleisli[IO, Int, Int]     = Kleisli.apply(_ =&gt; IO.pure(10))
  val processNumberIO: Kleisli[IO, Int, Int]     = Kleisli.apply(v =&gt; IO(v * 2))
  val writeDataToDBIO: Kleisli[IO, Int, Boolean] = Kleisli.apply(_ =&gt; IO.pure(true))

  val resultIO = getDataFromDbIO andThen processNumberIO andThen writeDataToDBIO
  resultIO.run(12)/*IO[Boolean]*/.unsafeRunSync() //Boolean</code></pre>

You can also use for-comprehension on Kleisli.

<pre><code class="scala">//For IO
  val getDataFromDbIO: Kleisli[IO, Int, Int]     = Kleisli.apply(_ =&gt; IO.pure(10))
  val processNumberIO: Kleisli[IO, Int, Int]     = Kleisli.apply(v =&gt; IO(v * 2))
  val writeDataToDBIO: Kleisli[IO, Int, Boolean] = Kleisli.apply(_ =&gt; IO.pure(true))

  val anotherFunction: Int =&gt; Kleisli[IO, Int, Int] = (v: Int) =&gt; Kleisli.liftF(processNumberIO(v))
  val oneMoreFunction: Int =&gt; Kleisli[IO, Int, Boolean] = (v: Int) =&gt;
    Kleisli.liftF(writeDataToDBIO(v))
  
  val resultOfFor: Kleisli[IO, Int, Boolean] = for {
    v  &lt;- getDataFromDbIO
    vv &lt;- anotherFunction(v)
    r  &lt;- oneMoreFunction(vv)
  } yield r
  resultOfFor(12).unsafeRunSync()</code></pre>

###### Conclusion:

Kleisli is just function **A=>F[B]** here F is Monad like Option, Try, Either, IO, Future(it is not completely monad but it is considered as monad). It enables the composition of functions that return a monadic value or monadic functions like A-> F[B].

**References** :  
<a href="https://typelevel.org/cats/datatypes/kleisli.html" target="_blank" rel="noopener">https://typelevel.org/cats/datatypes/kleisli.html</a>