---           
layout: post
title: Numerology calculations with F#
date: 2015-12-22 8:00 AM
updated: 2015-12-22 8:00 AM
comments: false
categories: fsharp
readmore: true
---

Disclaimer: Lets agree to treat Numerology as Magic and focus on the FSharp part. I pick this particular subject as good use case to apply a lot of what F# brings to the table. Please treat this as just some fun requirements to demonstrate F# types, functions and general functional programming.

We need to model the inputs to our library. So lets start with a Person. A Person has a name and date of birth. So lets start with a type person.

`type Person = ....`

The person type can be a tuple ..

`type Person = Name * DateOfBirth`

where Name and DateOfBirth are other types. 

Name has first name, middle name and last name. 

`type Name = string * string * string`

DateOfBirth has day, month and year.

`type DateOfBirth = int * int * int`

Now these are valid types; but we lose the context of each value in the tuple. So was it `DateOfBirth = firstname * middlename * lastname` or something else? To remove these questions; I thnk records is a better tool to model such data types.

```
type Person = {
	name : Name
	dob : DateOfBirth
} and
  Name = {
  	firstName : string
  	middleName : string
  	lastName : string
  }
  and 
  DateOfBirth = {
  	day : int
  	month : int
  	year : int
  }
```

So here is an example where we delcared F# records in a top-down fashion using the `and` keyword which makes it easier to read through. The `and` allows us to list the types after they are actually used. Also the names of the fields in the records carries the context around.

So a person can now be constructed as :

```
let person = { 
	name = { firstName="Krishna"; middleName="Bhargava"; lastName="Vangapandu"}; 
	dob = { day = 14; month = 1; year = 1985 } 
}
```

Now this is a little too cumbersome; so I want to make some factory methods that helps me build names and dobs so I can do something like:

``
let person = { 
	name = Name.build "Krishna Bhargava Vangapandu"; 
	dob = DateOfBirth.build (14,1,1985) 
}
``` 

We can do this by creating type extensions in F# for the `Name` type as:

```
type Name with
	    static member build (str : string) = 
	        let x = str.Split([| ' ' |])
	        let splits = x |> Array.length
	        if splits = 2 then 
	            { firstName = x.[0]; lastName = x.[1]; middleName = "" }
	        elif splits = 3 then 
	            { firstName = x.[0]; middleName = x.[1]; lastName = x.[2] }
	        else 
	            { firstName = x.[0]; middleName = ""; lastName = "" }
```

Similarly; we will create an extension type for `DateOfBirth` as:

```
type DateOfBirth with
    static member build (m, d, y) = 
    	{ day = d
          month = m
          year = y }
```

Now we have a `Person` and some helper functions to create person records. Note that as with most types in F#; records are immutable and you cannot change values of the record in-place. You can however create a new record with data from a previous record. For example:

```
let krishna = "Krishna Bhargava Vangapandu" |> Name.build
let archana = { krishna with firstName = "Archana"; middleName=""}
```

Lets start creating some numerology functions. We will group all the functions into a separate module called `Numbers`. 

The first function we will create is going to be called "destiny number". Before we go ahead with implementation of the destinyNumber function; lets talk a little about how names are converted to numbers; how numbers are reduced in general; so we have some requirements in mind.

```
Each character (a-z) has a numerical value between 1-9 associated with it. For example, A will be 1, B will be 2, ... I will be 9 and J will be again 1 ... and so on. We can do this as shown below. Once each character gets a numerical value; the numerical value of the name would be to sum these values and then again reduce the final number to a single digit. For example, if you get a sum as 23; you will reduce it to 5 as 23 -> 2 + 3 = 5. A simple way to do that would be to do a 23 % 9; you get the remainder as 5. 

But to make things slightly complex; not all numbers are to be reduced to single digits. For example, there are some compound numbers such as 11, 13, 14, 16, 19 and 22 which should be left as is. These are called Karmic Numbers. For example, if we use the above formula to compute the destiny number for "Krishna Bhargava Vangapandu"; the sum would be 16 which should not just be reduced to 7. To make matters further complicated; when you are combining two "numbers"; you will always reduce to single digit before you combine them. 

Now back to F#; we need a way to represent "number" which can either be single digit OR a karmic number. And as mentioned earlier; try to model everything you see as a type and then the structure flows naturally. In OO world; we would have modeled a root class - Number. Then derive two types out of Number - SingleNumber and KarmicNumber. In the functional world; for this case; you should think of "OR" types or "SUM" types or "discriminated unions" since we are talking of a type which can be one type or the other. 
```

```
type Number = 
	| Single of int
	| Karmic of int
```

*Side note*: We cannot place compile time constraints on `Single of int` such that only values between 1-9 are permitted. We can however create an enum in F# to represent the fixed domain of values for Single. There are, however, dependent types in some languages which permit such compile-time constraints.

Now that we have a type; the next natural (to me, highly opinionated) is to add type extensions for Number with some utility functions. For example; we need to convert an integer into Number; convert a Number back into integer; etc. To think of Number having a proper functional library is a good exercise. For example, consider you have Single 10; you would need a functor on it such you can multiply 10 with 2 and get back Single 20. Such functions can be part of this type extensions module. First lets look at the implementation to see what we need in the Numbers type extension.

```
module Numbers = 
    let reduce f (str : string) = 
        str.ToLower().ToCharArray()
        |> Array.filter f
        |> Array.map (fun c -> (c |> int) - 96) //reduce characters to 1-9
        |> Array.sum //sum up the numbers
        |> Number.convert //TODO:Need a int -> Number function
    
    (* destinyNumber : Name -> Number *)
    let destinyNumber (name : Name) =
    	(* demonstrates pattern matching *) 
        let { firstName = fname; middleName = mname; lastName = lname } = name
        let f = fun x -> true //always true function
        [| fname |> reduce f
           mname |> reduce f
           lname |> reduce f |]
        |> Array.reduce (Number.sum) //TODO: Need a function to sum Number -> Number -> Number
        |> Number.liftUp // TODO: Need a Number -> Number function for ensuring proper constraints.
```

The implementation is simple - take the first name, middle name, last name (see above on how we use pattern matching to extract the values from a record). Reduce each character to the assigned value between 1 & 9. Then sum up the values obtained. And look how F# reads exactly the same way as we expressed our algorithm. That is what functional programming brings to the table. In the imperative world; we would have been bothered with more responsibility where in we also express how we want to get the desired outcome of each step.

To focus on the missing functions; we need a way to convert the obtained numerical value to a Number. That is done by the convert function.
We need a way to combine two numbers; for now we will have a specialized method that just sums two numbers together. As mentioned earlier; when we combine each numbers; we need to push down the Karmic number as a Single number and then perform combinations only on Single numbers to get back another Number. But the final result in the step should still conform to the specifications of Single vs. Karmic number. Hence the need for liftUp. Lets see how we can apply all these constraints on our functions.

```
type Number with    
    static member private singleDigit i = 
        if i = 0 then 
            0 
        else
            let mod9 = i % 9
            if mod9 = 0 then 9
            else mod9
    
    static member convert (i : int) = 
        match i with
        | 11 | 13 | 14 | 16 | 19 | 22 -> Number.Karmic(i)
        | _ -> (Number.singleDigit i) |> Number.Single

    static member sum (Number.Single(m)) (Number.Single(n)) = 
        (m + n) |> Number.Single

    static member liftUp (Number.Single(x)) = x |> Number.convert
```
So in the above set of functions - sum and liftUp; the parameter type is still a Number and not really Number.Single. Number.Single is not a type as such but a case of our union type. Atleast we expressed it in a way that it shows we are only going to support Single cases of the Number. This is not necessarily an ideal way as there are no compiler checks on the caller to enforce Number.Single is only used to call the functions. If you want you can call the function with (Number.Karmic 11). The compiler will let it go but you will get a runtime MatchError.

```
Microsoft.FSharp.Core.MatchFailureException: The match cases were incomplete
   at FSI_0005.Number.sum(Number _arg3, Number _arg2)
   at <StartupCode$FSI_0008>.$FSI_0008.main@()
```
The compiler would generate a warning though. It tells you that you haven't really handled all the cases in your match expression which is exactly what Number.Single(x) in call is. At this time; unless the compiler can support dependent types; such a compiler time constraint cannot be enforced.

If you look back to the destinyNumber implementation; we compute Number for each of the names & then reduce it to a single Number passing the combinator Number.sum. However there is a bug here. Our reduce implementation will not always result in a Number.Single but can also give a Number.Karmic. But our sum function does not have a corresponding match on that. As mentioned earlier; we would want to convert the specified Number to always result in a Number.Single before we perform any form of combinations. So lets change our sum a little.

```
static member alwaysSingle n = 
        match n with
        | Number.Karmic(x) ->  x |> Number.singleDigit |> Number.Single
        | Number.Single(x) as s -> s

static member sum m n = 
    let internal_sum (Number.Single(x)) (Number.Single(y)) = 
        (x + y) |> Number.Single
    internal_sum (Number.alwaysSingle m) (Number.alwaysSingle n)
```

The above functions also demonstrate inner functions (internal_sum); pattern matching using `match with` expressions. 

Now that we have a sum function; what if we need a multiplication function? So lets refactor the function a little to build a combine function.

```
static member combine f m n = 
        let inner_combine (Number.Single(x)) (Number.Single(y)) = 
            (f x y) |> Number.Single
        inner_combine (Number.alwaysSingle m) (Number.alwaysSingle n)

static member sum = Number.combine (+)

static member diff = Number.combine (-)
```
Now we have a nice function `combine` that takes a function and two numbers; combines them using the function specified. We can then generate utility functions for sum and diff passing in the (+) and (-) as functions. Notice the usage of + and - operators as functions - which is what they really are.

With this, I would like to conclude the post. Hopefully, it showed my thought process around type-driven development and how your types can be expressed in F# and worked with. I am sure there are better explanations and corrections than what I managed to come up with. Feel free to send in PR for any corrections you want to suggest.
