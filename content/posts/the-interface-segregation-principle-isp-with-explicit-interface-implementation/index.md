+++
title = "The Interface Segregation Principle (ISP) with explicit interface implementation"
date = 2013-05-30T15:17:27Z
draft = false
tags = ["maintainability", "testability"]
categories = ["Software Design and Principles"]
+++

The project I am currently working on is a product development project. Our daily work is to add features (or fixing bugs) to our code base which has been around for over 10 years, this is why I’ve been encountering legacy code daily. As one might imagine, when the **maintainability and testability** of a project of this caliber is low-prioritized, refactoring becomes near impossible after a point. There are of course exceptions to this, some subsystems do renew themselves periodically, lucky for them!

I was reading a namespace to hunt a bug and I have discovered a well implemented practice to achieve maintainability and testability to some degree with the **Interface segregation principle (ISP)**. Every time I hit the F12 (Go To Definition) on a method or on a property, I was jumping to an Interface file. I had to find all the usages with Shift+F12 (Find Symbols) and locate implementations. This isn’t something that I am used to seeing every day. I took a look and found out that all the classes implement the interfaces **explicitly**. Therefore I was always routed to interface file during “Go To Definition”.

![Clark](Clark.png)

At first I thought explicit interface implementation unnecessary, because it prevented me from exploring the code. After I decided to write unit test first upon a change-request, I realized how wrong I had been. Like I’ve mentioned, the system was bulky and aged and it was getting harder to write a unit test for it without mocking. But for this subsystem it wasn’t the case, as it's mainly programed to an interface and all interface implementations were explicit. This is basically how you stick with the ISP. 

This subsystem is actually playing the role of a minor framework to several smaller subsystems. The users of this subsystems are only familiar with the interfaces, thus allowing us to refactor or apply non-breaking-changes without touching the client code, just because client-codes are forced to program to interfaces.

I’ll try to explain all this with a different example. Take Superman, Clark Kent being his cover, in his world not many people know about his real identity and none of them expect him to save a plane from crashing.

Take the following classes,

For the **Terran**, we’ll define a *Name* property and *Run()* for method.

```csharp
interface ITerran
{ 
	string Name { get; set; } 
	void Run(); 
}
```

For the *Kryptonian* we’ll define again a *Name, Run() and Fly()*.

```csharp
interface IKryptonian
{ 
	string Name { get; set; } 
	void Fly(); 
	void Run(); 
}
```

Naturally the *Person* class will be the default class for our implementation.

```csharp
abstract class Person 
{ 
	protected string Name { get; set; }
 
	public Person(string name) 
	{ 
		this.Name = name; 
	}
}
```

I’ve added a private *Name* property to the *Person* abstract class. As you can see the *Name* can be set in constructor.

Below you can observe the Superman class implementing both *ITerran* and *IKryptonian* interfaces.

Notice how the protected *Name* property is mapped to the public property of our *IKryptonian*.

A client code using the *Person* class can’t use any property or methods without casting to *IKryptonian*. As a result the **client code is forced to program to an interface** meaning instead of using the *Person* class itself, it uses the *IKyrptonian*.

```csharp
sealed class Superman : Person, ITerran, IKryptonian 
{ 
	public Superman(string name) : base(name) { } 
	
	string IKryptonian.Name 
	{ 
		get { return this.Name; } 
		set { this.Name = value; } 
	} 
	
	string ITerran.Name { get; set; } 
	
	void IKryptonian.Fly() 
	{ 
		Trace.WriteLine("It's a Bird... It's a Plane... It's Superman"); 
	} 

	void IKryptonian.Run() 
	{ 
		Trace.WriteLine(String.Format("{0} is running superfast", ((IKryptonian)this).Name)); 
	} 
	
	void ITerran.Run() 
	{ 
		Trace.WriteLine(String.Format("{0} is jogging", ((ITerran)this).Name)); 
	} 	
}
```

In other words, the ones who interact with Clark, expect him to be the bespectacled nerdy journalist and don’t expect any superhero action from him.  

```csharp
public static void Example() 
{ 
	IKryptonian kalEl = GetKryptonianWithGivenName("Kal-El"); 
	ITerran clark = kalEl as ITerran; 
	
	if (clark != null) 
	{ 
		clark.Name = "Clark Kent"; 
		Trace.WriteLine(String.Format("Human name : {0}", clark.Name)); 
		clark.Run(); 
	} 
	
	if (kalEl != null) 
	{ 
		Trace.WriteLine(String.Format("Kryptonian name : {0}", kalEl.Name)); 
		kalEl.Run(); 
		kalEl.Fly(); 
	} 
} 

private static IKryptonian GetKryptonianWithGivenName(string givenName) 
{ 
	return new Superman(givenName); 
}
```

When creating the *Superman* class we discreetly give it a first name (*Kal-El*). Subsequently we convert a reference from this object, and name it Clark Kent.  We don't expect from a Terran to fly, similarly we cast to relevant class before we ask his name. Even though this way of thinking might take a little time to get used to when aimed for programming, in time you will see its benefits, some of which I will try to list below. Basically **you should think in terms of contracts, not implementations**.

BTW, this is the result of the program:

```ini
Human name : Clark Kent 
Clark Kent is jogging. 
Kryptonian name : Kal-El 
Kal-El is running superfast! 

It's a Bird... It's a Plane... It's Superman!
```

At the beginning of this article I’ve explained how this approach has helped me with unit testing. As you can see in the example method, I created my object using *GetKryptonianWithGivenName*().  As much as it is hard-coded, we should remember that it is a composition via Concrete Factory. So I can change this method during System-Under-Test (SUT). For instance, if I wanted to change the Superman class with a fake, all I had to do would be to create a *Person* class that implements the *IKryptonian*.  

```csharp
sealed class SupermanStub : Person, ITerran, IKryptonian 
{ 
	public SupermanStub(string name) : base("Fake of " + name) { } 
	string IKryptonian.Name 
	{ 
		get { return this.Name; } 
		set { this.Name = value; } 
	} 

	void IKryptonian.Fly() 
	{ 
		Trace.WriteLine("I am a fake, I can't fly. :/"); 
	} 

	void IKryptonian.Run() 
	{ 
		Trace.WriteLine("I might be a fake, but I can run."); 
	} 

	void ITerran.Run() 
	{ 
		Trace.WriteLine(String.Format("Running like a human is easy to fake ;).", ((ITerran)this).Name)); 
	} 
	
	string ITerran.Name { get; set; } 
}
```

As you can see it was easy to create a stub for SUT using the interfaces like below. If the classes weren’t programmed to interface, this wouldn't be possible and I would have to mock the Superman class.  

```csharp
private static IKryptonian GetKryptonianWithGivenName(string givenName) 
{
	return new SupermanStub(givenName); 
}
```

```csharp
Human name: Clark Kent 
Running like a human is easy to fake ;). 
Kryptonian name: Fake of Kal-El 
I might be a fake, but I can run. 
I am a fake, I can't fly. :/
```

Let's assume that the *Run*() method of the *IKryptonian* became an unnecessary feature since nobody expect to see Superman run to save the world instead of flying. As a result, the design team will decide to remove this *Run*() method from the *IKryptonian* interface. First, the base team will define this method as obsolete. This allows the client codes (*Superman* class) to remove their implantations for this method. After expiration of given time, the *Run*() method will be deleted from *IKryptonian* interface. If really the client codes did remove their implementations, build process won't be a problem and the unnecessary *Run*() method would be removed from the system once and for all. But, if client codes did implement the *IKryptonian* interface implicitly there would be no build problem even they didn't delete their *Run*() implementation. And yet there will be unused-code.

On the down-side, I can only repeat the problem of code-navigation in Visual Studio as I tried to brief my experience above in the first paragraphs. But, if you have *ReSharper*, there is useful shortcuts for navigating to implementation (Go to implementation).

In the Superman example there is no possibility to implement both interfaces (*ITerran* and *IKryptonian*) implicitly, because of the ambiguity (some properties and methods have same names). The explicit interface implementation is commonly being used to overcome this interface ambiguity problem. But, we tried to use the explicit implementation for the sake of maintainability and testability, nirvana for the ISP.

Further readings :  

  
-   Specially **Chapter 1** for "Program to an interface, not an implementation" [Design Patterns : Elements of reusable object oriented software](http://www.amazon.co.uk/dp/0201633612/ref=as_li_ss_til?tag=publicfunctio-21&camp=2902&creative=19466&linkCode=as4&creativeASIN=0201633612&adid=16HNSQN0SPHYZ5WD2K6R&&ref-refURL=http%3A%2F%2Frcm-uk.amazon.co.uk%2Fe%2Fcm%3Flt1%3D_blank%26bc1%3D000000%26IS2%3D1%26bg1%3DFFFFFF%26fc1%3D000000%26lc1%3D0000FF%26t%3Dpublicfunctio-21%26o%3D2%26p%3D8%26l%3Das4%26m%3Damazon%26f%3Difr%26ref%3Dss_til%26asins%3D0201633612) - GoF
  
-   [Interface Segregation Principle](http://www.objectmentor.com/resources/articles/isp.pdf) - A chapter from Robert C. Martin's book. (PDF)
  
-   [C#: Interfaces - Implicit and Explicit implementation](http://stackoverflow.com/questions/143405/c-interfaces-implicit-and-explicit-implementation) - Stackoverflow Q&A
