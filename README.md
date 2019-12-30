# 31-days-refactoring
This will keep reminding me refactoring my silly code

## Contents

- [Refactoring Day 1 : Encapsulate Collection](README.md#refactoring-day-1--encapsulate-collection)
- [Refactoring Day 2 : Move Method](README.md#refactoring-day-2--move-method)
- [Refactoring Day 3 : Pull Up Method](README.md#refactoring-day-3--pull-up-method)
- [Refactoring Day 4 : Push Down Method](README.md#refactoring-day-4--push-down-method)
- [Refactoring Day 5 : Pull Up Field](README.md#refactoring-day-5--pull-up-field)
- [Refactoring Day 6 : Push Down Field](README.md#refactoring-day-6--push-down-field)
- [Refactoring Day 7 : Rename (method, class, parameter)](README.md#refactoring-day-7--rename-method-class-parameter)
- [Refactoring Day 8 : Replace Inheritance with Delegation](README.md#refactoring-day-8--replace-inheritance-with-delegation)
- [Refactoring Day 9 : Extract Interfaces](README.md#refactoring-day-9--extract-interfaces)
- [Refactoring Day 10 : Extract Method](README.md#refactoring-day-10--extract-method)
- [Refactoring Day 11 : Switch to Strategy](README.md#refactoring-day-11--switch-to-strategy)
- [Refactoring Day 12 : Break Dependencies](README.md#refactoring-day-12--break-dependencies)
- [Refactoring Day 13 : Extract Method Object](README.md#refactoring-day-13--extract-method-object)
- [Refactoring Day 14 : Break Responsibilities](README.md#refactoring-day-14--break-responsibilities)
- [Refactoring Day 15 : Remove Duplication](README.md#refactoring-day-15--remove-duplication)
- [Refactoring Day 16 : Encapsulate Conditional](README.md#refactoring-day-16--encapsulate-conditional)
- [Refactoring Day 17 : Extract Superclass](README.md#refactoring-day-17--extract-superclass)
- [Refactoring Day 18 : Replace exception with conditional](README.md#refactoring-day-18--replace-exception-with-conditional)
- [Refactoring Day 19 : Extract Factory Class](README.md#refactoring-day-19--extract-factory-class)
- [Refactoring Day 20 : Extract Subclass](README.md#refactoring-day-20--extract-subclass)
- [Refactoring Day 21 : Collapse Hierarchy](README.md#refactoring-day-21--collapse-hierarchy)
- [Refactoring Day 22 : Break Method](README.md#refactoring-day-22--break-method)
- [Refactoring Day 23 : Introduce Parameter Object](README.md#refactoring-day-23--introduce-parameter-object)
- [Refactoring Day 24 : Remove Arrowhead Antipattern](README.md#refactoring-day-24--remove-arrowhead-antipattern)
- [Refactoring Day 25 : Introduce Design By Contract checks](README.md#refactoring-day-25--introduce-design-by-contract-checks)
- [Refactoring Day 26 : Remove Double Negative](README.md#refactoring-day-26--remove-double-negative)
- [Refactoring Day 27 : Remove God Classes](README.md#refactoring-day-27--remove-god-classes)
- [Refactoring Day 28 : Rename boolean method](README.md#refactoring-day-28--rename-boolean-method)

---

## Refactoring Day 1 : Encapsulate Collection

* Not expose a full collection to consumers of a class
* Only expose the collection as something you can iterate over without modifying the collection
* Using this can ensure that consumers do not mis-use your collection and introduce bugs into the code

```C#
public class Order
{
    private List<OrderLine> _orderLines;

    public IEnumerable<OrderLine> OrderLines
    {
        get { return _orderLines; }
    }

    public void AddOrderLine(OrderLine orderLine)
    {
        _orderTotal += orderLine.Total;
        _orderLines.Add(orderLine);
    }

    public void RemoveOrderLine(OrderLine orderLine)
    {
        orderLine = _orderLines.Find(o => o == orderLine);
        if (orderLine == null) return;

        _orderTotal -= orderLine.Total
        _orderLines.Remove(orderLine);
    }
}
```

## Refactoring Day 2 : Move Method

**Move a method to a better location**

Before refactoring:

```C#
public class BankAccount
{
    public BankAccount(int accountAge, int creditScore,
                            AccountInterest accountInterest)
    {
        AccountAge = accountAge;
        CreditScore = creditScore;
        AccountInterest = accountInterest;
    }

    public double CalculateInterestRate()
    {
        if (Account.CreditScore > 800)
            return 0.02;

        if (Account.AccountAge > 10)
            return 0.03;

        return 0.05;
    }
}

public class AccountInterest
{
    public BankAccount Account { get; private set; }

    public AccountInterest(BankAccount account)
    {
        Account = account;
    }

    public double InterestRate
    {
        get { return Account.CalculateInterestRate(); }
    }

    public bool IntroductoryRate
    {
        get { return Account.CalculateInterestRate() < 0.05; }
    }
}   
```

After refactoring:

```C#
public class BankAccount
{
    public BankAccount(int accountAge, int creditScore,
                            AccountInterest accountInterest)
    {
        AccountAge = accountAge;
        CreditScore = creditScore;
        AccountInterest = accountInterest;
    }

    public int AccountAge { get; private set; }
    public int CreditScore { get; private set; }
    public AccountInterest AccountInterest { get; private set; }
}

public class AccountInterest
{
    public BankAccount Account { get; private set; }

    public AccountInterest(BankAccount account)
    {
        Account = account;
    }

    public double InterestRate
    {
        get { return CalculateInterestRate(); }
    }

    public bool IntroductoryRate
    {
        get { return CalculateInterestRate() < 0.05; }
    }

    public double CalculateInterestRate()
    {
        if (Account.CreditScore > 800)
            return 0.02;

        if (Account.AccountAge > 10)
            return 0.03;

        return 0.05;
    }
}
```

The point of interest here is the `BankAccount.CalculateInterest` method. A hint that you need the Move Method refactoring is **_when another class is using a method more often then the class in which it lives, then it makes sense to move the method to the class where it is primarily used_**

## Refactoring Day 3 : Pull Up Method

The Pull Up Method refactoring is the process of **taking a method and “Pulling” it up in the inheritance chain**. This is used when a method needs to be used by multiple implementers.

Before refactoring:

```C#
public abstract class Vehicle
{
    // other methods
}

public class Car : Vehicle
{
    public void Turn(Direction direction)
    {
        // code here
    }
}

public class Motorcycle : Vehicle
{
}

public enum Direction
{
    Left,
    Right
}
```

After refactoring:

```C#
public abstract class Vehicle
{
    public void Turn(Direction direction)
    {
        // code here
    }


public class Car : Vehicle
{
}

public class Motorcycle : Vehicle
{
}

public enum Direction
{
    Left,
    Right
}
```

Some notes:

* The only drawback is we have increased surface area of the base class adding to it’s complexity so use wisely. Only place methods that need to be used by more that one derived class.
* Once you start overusing inheritance it breaks down pretty quickly and you should start to lean towards composition over inheritance.

## Refactoring Day 4 : Push Down Method

Before refactoring:

```C#
public abstract class Animal
{
    public void Bark()
    {
        // code to bark
    }
}

public class Dog : Animal
{
}

public class Cat : Animal
{
}
```

Perhaps at one time our cat could bark, but now we no longer need that functionality on the Cat class. So we **“Push Down”** the Bark method into the Dog class as it is no longer needed on the base class but perhaps it is still needed when dealing explicitly with a Dog

After refactoring:

```C#
public abstract class Animal
{
}

public class Dog : Animal
{
    public void Bark()
    {
        // code to bark
    }
}

public class Cat : Animal
{
}
```

_If there is not any behavior still located on the Animal base class, we cab turn the Animal abstract class into an interface instead as no code is required on the contract and can be treated as a marker interface._

## Refactoring Day 5 : Pull Up Field

Before refactoring:

```C#
public abstract class Account
{
}
   
public class CheckingAccount : Account
{
    private decimal _minimumCheckingBalance = 5m;
}

public class SavingsAccount : Account
{
    private decimal _minimumSavingsBalance = 5m;
}
```

After refactoring:

```C#
public abstract class Account
{
    protected decimal _minimumBalance = 5m;
}

public class CheckingAccount : Account
{
}

public class SavingsAccount : Account
{
}
```

## Refactoring Day 6 : Push Down Field

Before refactoring:

```C#
public abstract class Task
    {
    protected string _resolution;
}

public class BugTask : Task
{
}

public class FeatureTask : Task
{
}
```

After refactoring:

```C#
public abstract class Task
{
}

public class BugTask : Task
{
    private string _resolution;
}

public class FeatureTask : Task
{
}
```

We have a string field that is only used by one derived class, and thus can be pushed down as no other classes are using it.

## Refactoring Day 7 : Rename (method, class, parameter)

All too often we do not name methods/classes/parameters properly that leads to a misunderstanding as to what the method/class/parameter’s function is. When this occurs, assumptions are made and bugs are introduced to the system

Before refactoring:

```C#
public class Person
{
    public string FN { get; set; }
   
    public decimal ClcHrlyPR()
    {
        // code to calculate hourly payrate
        return 0m;
    }
}
```

As you can see here, we have a class/method/parameter that all have very non-descriptive, obscure names. They can be interpreted in a number of different ways.

Applying this refactoring is as simple as renaming the items at hand to be more descriptive and convey what exactly they do.

After refactoring:

```C#
// Changed the class name to Employee
public class Employee
{
    public string FirstName { get; set; }
   
    public decimal CalculateHourlyPay()
    {
        // code to calculate hourly payrate
        return 0m;
    }
}
```

## Refactoring Day 8 : Replace Inheritance with Delegation

* Inheritance should only be used in logical circumstances but it is often used for convenience purposes.
* Inheritance should ONLY be used for scenarios where inheritance is warranted. Not instances where it makes it quicker to throw down code.

Before refactoring:

```C#
public class Sanitation
{
    public string WashHands()
    {
        return "Cleaned!";
    }
}

public class Child : Sanitation
{
}
```

After refactoring:

```C#
public class Sanitation
{
    public string WashHands()
    {
        return "Cleaned!";
    }
}

public class Child
{
    private Sanitation Sanitation { get; set; }
  
    public Child()
    {
        Sanitation = new Sanitation();
    }

    public string WashHands()
    {
        return Sanitation.WashHands();
    }
}
```

## Refactoring Day 9 : Extract Interfaces

Before refactoring:

> When you notice more than one class using a similar subset of methods on a class, it is useful to break the dependency and introduce an interface that the consumers to use.

```C#
public class ClassRegistration
{
    public void Create()
    {
        // create registration code
    }
   
    public void Transfer()
    {
        // class transfer code
    }

    public decimal TotalTotal { get; private set; }
}

public class RegistrationProcessor
{
    public decimal ProcessRegistration(ClassRegistration registration)
    {
        registration.Create();
        return registration.Total;
    }
}
```

After refactring:

> Extracted the methods that both consumers use and placed them in an interface. Now the consumers don’t care/know about the class that is implementing these methods. We have decoupled our consumer from the actual implementation and depend only on the contract that we have created.

```C#
public interface IClassRegistration
{
    void Create();
    decimal Total { get; }
}

public class ClassRegistration : IClassRegistration
{
    public void Create()
    {
        // create registration code
    }

    public void Transfer()
    {
        // class transfer code
    }

    public decimal Total { get; private set; }
}

public class RegistrationProcessor
{
    public decimal ProcessRegistration(IClassRegistration registration)
    {
        registration.Create();
        return registration.Total;
    }
}
```

## Refactoring Day 10 : Extract Method

> Make code more readable by placing logic behind descriptive method names.

Before refactoring:

```C#
public class Receipt
{
    private IList<decimal> Discounts { get; set; }
    private IList<decimal> ItemTotals { get; set; }
   
    public decimal CalculateGrandTotal()
    {
        decimal subTotal = 0m;
        foreach (decimal itemTotal in ItemTotals)
            subTotal += itemTotal;

        if (Discounts.Count > 0)
        {
            foreach (decimal discount in Discounts)
                subTotal -= discount;
        }

        decimal tax = subTotal * 0.065m;
  
        subTotal += tax;
  
        return subTotal;
    }
}
```

After refactoring:

The `CalculateGrandTotal` method is actually doing three different things here:

* Calculating the subtotal
* Applying any discounts
* Calculating the tax for the receipt

```C#
public class Receipt
{
    private IList<decimal> Discounts { get; set; }
    private IList<decimal> ItemTotals { get; set; }

    public decimal CalculateGrandTotal()
    {
        decimal subTotal = CalculateSubTotal();
        
        subTotal = CalculateDiscounts(subTotal);
        
        subTotal = CalculateTax(subTotal);
        
        return subTotal;
    }

    private decimal CalculateTax(decimal subTotal)
    {
        decimal tax = subTotal * 0.065m;
    
        subTotal += tax;
    
        return subTotal;
    }
    
    private decimal CalculateDiscounts(decimal subTotal)
    {
        if (Discounts.Count > 0)
        {
            foreach (decimal discount in Discounts)
                subTotal -= discount;
        }
        return subTotal;
    }

    private decimal CalculateSubTotal()
    {
        decimal subTotal = 0m;
        foreach (decimal itemTotal in ItemTotals)
            subTotal += itemTotal;
        return subTotal;
    }
}
```

## Refactoring Day 11 : Switch to Strategy

This refactoring is used when you have a larger switch statement that continually changes because of new conditions being added. In these cases it’s often better to introduce the strategy pattern and encapsulate each condition in it’s own class.

Before refactoring:

```C#
namespace LosTechies.DaysOfRefactoring.SwitchToStrategy.Before
{
    public class ClientCode
    {
        public decimal CalculateShipping()
        {
            ShippingInfo shippingInfo = new ShippingInfo();
            return shippingInfo.CalculateShippingAmount(State.Alaska);
        }
    }
    
    public enum State
    {
        Alaska,
        NewYork,
        Florida
    }

    public class ShippingInfo
    {
        public decimal CalculateShippingAmount(State shipToState)
        {
            switch(shipToState)
            {
                case State.Alaska:
                    return GetAlaskaShippingAmount();
                case State.NewYork:
                    return GetNewYorkShippingAmount();
                case State.Florida:
                    return GetFloridaShippingAmount();
                default:
                    return 0m;
            }
        }

        private decimal GetAlaskaShippingAmount()
        {
            return 15m;
        }

        private decimal GetNewYorkShippingAmount()
        {
            return 10m;
        }

        private decimal GetFloridaShippingAmount()
        {
            return 3m;
        }
    }
}
```

After refactoring:

### Approach 1

Passing the enum as the dictionary key, we can select the proper implementation and execute the code at hand.

In the future when you want to add another condition, add another implementation and add the implementation to the ShippingCalculations dictionary.

```C#
using System.Collections.Generic;

namespace LosTechies.DaysOfRefactoring.SwitchToStrategy.After
{
    public class ClientCode
    {
        public decimal CalculateShipping()
        {
            ShippingInfo shippingInfo = new ShippingInfo();
            return shippingInfo.CalculateShippingAmount(State.Alaska);
        }
    }

    public enum State
    {
        Alaska,
        NewYork,
        Florida
    }

    public class ShippingInfo
    {
        private IDictionary<State, IShippingCalculation> ShippingCalculations { get; set; }
        
        public ShippingInfo()
        {
            ShippingCalculations = new Dictionary<State, IShippingCalculation>
            {
                { State.Alaska, new AlaskShippingCalculation() },
                { State.NewYork, new NewYorkShippingCalculation() },
                { State.Florida, new FloridaShippingCalculation() }
            }; 
        }

        public decimal CalculateShippingAmount(State shipToState)
        {
            return ShippingCalculations[shipToState].Calculate();
        }
    }

    public interface IShippingCalculation
    {
        decimal Calculate();
    }
    
    public class AlaskShippingCalculation : IShippingCalculation
    {
        public decimal Calculate()
        {
            return 15m;
        }
    }

    public class NewYorkShippingCalculation : IShippingCalculation
    {
        public decimal Calculate()
        {
            return 10m;
        }
    }

    public class FloridaShippingCalculation : IShippingCalculation
    {
        public decimal Calculate()
        {
            return 3m;
        }
    }
}
```

### Approach 2

The enum for the state now lives in the strategy and ninject gives us a IEnumerable of all bindings to the constructor of IShippingInfo. 

We then create a dictionary using the state property on the strategy to populate our dictionary and the rest is the same.

```C#
public interface IShippingInfo
{
    decimal CalculateShippingAmount(State state);
}

public class ClientCode
{
    [Inject]
    public IShippingInfo ShippingInfo { get; set; }

    public decimal CalculateShipping()
    {
        return ShippingInfo.CalculateShippingAmount(State.Alaska);
    }
}

    public enum State
    {
        Alaska,
        NewYork,
        Florida
    }
  
    public class ShippingInfo : IShippingInfo
    {
        private IDictionary<State, IShippingCalculation> ShippingCalculations { get; set; }

        public ShippingInfo(IEnumerable<IShippingCalculation> shippingCalculations)
        {
            ShippingCalculations = shippingCalculations.ToDictionary(
                                          calc => calc.State);
        }

        public decimal CalculateShippingAmount(State shipToState)
        {
            return ShippingCalculations[shipToState].Calculate();
        }
    }

    public interface IShippingCalculation
    {
        State State { get; }
        decimal Calculate();
    }

    public class AlaskShippingCalculation : IShippingCalculation
    {
        public State State { get { return State.Alaska; } }

        public decimal Calculate()
        {
            return 15m;
        }
    }

    public class NewYorkShippingCalculation : IShippingCalculation
    {
        public State State { get { return State.NewYork; } }

        public decimal Calculate()
        {
            return 10m;
        }
    }

    public class FloridaShippingCalculation : IShippingCalculation
    {
        public State State { get { return State.Florida; } }

        public decimal Calculate()
        {
            return 3m;
        }
    }
```

## Refactoring Day 12 : Break Dependencies

In this example we have client code that is using a static class to accomplish some work.

The problem with this when it comes to unit testing because there is no way to mock the static class from our unit test.

To work around this you can apply a wrapper interface around the static to create a seam and break the dependency on the static.

Before refactoring:

```C#
public class AnimalFeedingService
{
    private bool FoodBowlEmpty { get; set; }

    public void Feed()
    {
        if (FoodBowlEmpty)
            Feeder.ReplenishFood();

        // more code to feed the animal
    }
}

public static class Feeder
{
    public static void ReplenishFood()
    {
        // fill up bowl
    }        
}
```

All we did to apply this refactoring was introduce an interface and class that simply calls the underlying static class. So the behavior is still the same, just the manner in which it is invoked has changed. This is good to get a starting point to begin refactoring from and an easy way to add unit tests to your code base.

After refactoring:

```C#
public class AnimalFeedingService
{
    public IFeederService FeederService { get; set; }

    public AnimalFeedingService(IFeederService feederService)
    {
        FeederService = feederService;
    }
   
    private bool FoodBowlEmpty { get; set; }

    public void Feed()
    {
        if (FoodBowlEmpty)
            FeederService.ReplenishFood();

        // more code to feed the animal
    }
}

public interface IFeederService
{
    void ReplenishFood();
}

public class FeederService : IFeederService
{
    public void ReplenishFood()
    {
        Feeder.ReplenishFood();
    }
}

public static class Feeder
{
    public static void ReplenishFood()
    {
        // fill up bowl
    }
}
```

We can now mock IFeederService during our unit test via the AnimalFeedingService constructor by passing in a mock of IFeederService. Later we can move the code in the static into FeederService and delete the static class completely once we have some tests in place.

## Refactoring Day 13 : Extract Method Object

Before refactoring:

```C#
public class OrderLineItem
{
    public decimal Price { get; private set; }
}

public class Order
{
    private IList<OrderLineItem> OrderLineItems { get; set; }
    private IList<decimal> Discounts { get; set; }
    private decimal Tax { get; set; }

    public decimal Calculate()
    {
        decimal subTotal = 0m;

        // Total up line items
        foreach (OrderLineItem lineItem in OrderLineItems)
        {
            subTotal += lineItem.Price;
        }

        // Subtract Discounts
        foreach (decimal discount in Discounts)
            subTotal -= discount;

        // Calculate Tax
        decimal tax = subTotal * Tax;

        // Calculate GrandTotal
        decimal grandTotal = subTotal + tax;

        return grandTotal;
    }
}
```

This entails passing a reference to the class that will be returning the computation to a new object that has the multiple methods via the constructor

After refactoring:

```C#
public class OrderLineItem
{
    public decimal Price { get; private set;}
}
    
public class Order
{
    public IEnumerable<OrderLineItem> OrderLineItems { get; private set;}
    public IEnumerable<decimal> Discounts { get; private set; }
    public decimal Tax { get; private set; }
  
    public decimal Calculate()
    {
        return new OrderCalculator(this).Calculate();
    }
}

public class OrderCalculator
{
    private decimal SubTotal { get; set;}
    private IEnumerable<OrderLineItem> OrderLineItems { get; set; }
    private IEnumerable<decimal> Discounts { get; set; }
    private decimal Tax { get; set; }

    public OrderCalculator(Order order)
    {
        OrderLineItems = order.OrderLineItems;
        Discounts = order.Discounts;
        Tax = order.Tax;
    }

    public decimal Calculate()
    {
        CalculateSubTotal();

        SubtractDiscounts();

        CalculateTax();

        return SubTotal;
    }

    private void CalculateSubTotal()
    {
        // Total up line items
        foreach (OrderLineItem lineItem in OrderLineItems)
            SubTotal += lineItem.Price;
    }

    private void SubtractDiscounts()
    {
        // Subtract Discounts
        foreach (decimal discount in Discounts)
            SubTotal -= discount;
    }

    private void CalculateTax()
    {
        // Calculate Tax
        SubTotal += SubTotal * Tax;
    }
}
```

## Refactoring Day 14 : Break Responsibilities

Before refactoring:

```C#
public class Video
{
    public void PayFee(decimal fee)
    {
    }
   
    public void RentVideo(Video video, Customer customer)
    {
        customer.Videos.Add(video);
    }
  
    public decimal CalculateBalance(Customer customer)
    {
        return customer.LateFees.Sum();
    }
}

public class Customer
{
    public IList<decimal> LateFees { get; set; }
    public IList<Video> Videos { get; set; }
}
```

As you can see here, the Video class has two responsibilities, once for handling video rentals, and another for managing how many rentals a customer has. We can break out the customer logic into it’s own class to help seperate the responsibilities.

```C#
public class Video
{
    public void RentVideo(Video video, Customer customer)
    {
        customer.Videos.Add(video);
    
}

public class Customer
{
    public IList<decimal> LateFees { get; set; }
    public IList<Video> Videos { get; set; }

    public void PayFee(decimal fee)
    {
    }

    public decimal CalculateBalance(Customer customer)
    {
        return customer.LateFees.Sum();
    }
}
```

## Refactoring Day 15 : Remove Duplication

* This is probably one of the most used refactoring in the forms of methods that are used in more than one place
* It is often added to the codebase through laziness or a developer that is trying to produce as much code as possible, as quickly as possible

```C#
public class MedicalRecord
{
    public DateTime DateArchived { get; private set; }
    public bool Archived { get; private set; }

    public void ArchiveRecord()
    {
        Archived = true;
        DateArchived = DateTime.Now;
    }

    public void CloseRecord()
    {
        Archived = true;
        DateArchived = DateTime.Now;
    }
}
```

We move the duplicated code to a shared method and voila! No more duplication. Please enforce this refactoring whenever possible. It leads to much fewer bugs because you aren’t copy/pasting the bugs throughout the code.

```C#
public class MedicalRecord
{
    public DateTime DateArchived { get; private set; }
    public bool Archived { get; private set; }

    public void ArchiveRecord()
    {
        SwitchToArchived();
    }

    public void CloseRecord()
    {
        SwitchToArchived();
    }

    private void SwitchToArchived()
    {
        Archived = true;
        DateArchived = DateTime.Now;s
    }
}
```

## Refactoring Day 16 : Encapsulate Conditional

Sometimes when doing a number of different checks within a conditional the intent of what you are testing for gets lost in the conditional.

```C#
public class RemoteControl
{
    private string[] Functions { get; set; }
    private string Name { get; set; }
    private int CreatedYear { get; set; }
   
    public string PerformCoolFunction(string buttonPressed)
    {
        // Determine if we are controlling some extra function
        // that requires special conditions
        if (Functions.Length > 1 && Name == "RCA" &&
                                CreatedYear > DateTime.Now.Year - 2)
            return "doSomething";
    }
}
```

After we apply the refactoring, you can see the code reads much easier and conveys intent:

```C#
public class RemoteControl
{
    private string[] Functions { get; set; }
    private string Name { get; set; }
    private int CreatedYear { get; set; }
   
    private bool HasExtraFunctions
    {
        get { return Functions.Length > 1 && Name == "RCA" &&
                                CreatedYear > DateTime.Now.Year - 2; }
    }

    public string PerformCoolFunction(string buttonPressed)
    {
        // Determine if we are controlling some extra function
        // that requires special conditions
        if (HasExtraFunctions)
            return "doSomething";
    }
}
```

## Refactoring Day 17 : Extract Superclass

This refactoring is used quite often when you have a number of methods that you want to “pull up” into a base class to allow other classes in the same hierarchy to use.

Before refactoring:

```C#
public class Dog
{
    public void EatFood()
    {
        // eat some food
    }
   
    public void Groom()
    {
        // perform grooming
    }
}
```

After applying the refactoring we just move the required methods into a new base class. This is very similar to the [pull up refactoring], except that you would apply this refactoring when a base class doesn’t already exist.

```C#
public class Animal
{
    public void EatFood()
    {
        // eat some food
    }
   
    public void Groom()
    {
        // perform grooming
    }
}

public class Dog : Animal
{
}
```

## Refactoring Day 18 : Replace exception with conditional

A common code smell that I come across from time to time is using exceptions to control program flow. You may see something to this effect:

```C#
public class Microwave
{
    private IMicrowaveMotor Motor { get; set;}
   
    public bool Start(object food)
    {
        bool foodCooked = false;
        try
        {
            Motor.Cook(food);
            foodCooked = true;
        }
        catch(InUseException)
        {
            foodcooked = false;
        }    
    }
    
    return foodCooked;
}
```

Most of the time you can replace this type of code with a proper conditional and handle it properly.

```C#
public class Microwave
{
    private IMicrowaveMotor Motor { get; set; }

    public bool Start(object food)
    {
        if (Motor.IsInUse)
            return false;

        Motor.Cook(food);
    
        return true;
    }
}
```

## Refactoring Day 19 : Extract Factory Class

```C#
public class PoliceCarController
{
    public PoliceCar New(int mileage, bool serviceRequired)
    {
        PoliceCar policeCar = new PoliceCar();
        policeCar.ServiceRequired = serviceRequired;
        policeCar.mileage = mileage;

        return policeCar;
    }
}
```

As we can see, the new action is responsible for creating a PoliceCar and then setting some initial properties on the police car depending on some external input. This works fine for simple setup, but over time this can grow and the burden of creating the police car is put on the controller which isn’t really something that the controller should be tasked with. In this instance we can extract our creation code and place in a Factory class that has the distinct responsibility of create instances of PoliceCar’s

```C#
public interface IPoliceCarFactory
{
    PoliceCar Create(int mileage, bool serviceRequired);
}

public class PoliceCarFactory : IPoliceCarFactory
{
    public PoliceCar Create(int mileage, bool serviceRequired)
    {
        PoliceCar policeCar = new PoliceCar();
        policeCar.ServiceRequired = serviceRequired;
        policeCar.Mileage = mileage;
        return policeCar;
    }
}

public class PoliceCarController
{
    public IPoliceCarFactory PoliceCarFactory { get; set; }

    public PoliceCarController(IPoliceCarFactory policeCarFactory)
    {
        PoliceCarFactory = policeCarFactory;
    }

    public PoliceCar New(int mileage, bool serviceRequired)
    {
        return PoliceCarFactory.Create(mileage, serviceRequired);
    }
}
```

Now that we have the creation logic put off to a factory, we can add to that one class that is tasked with creating instances for us without the worry of missing something during setup or duplicating code.

## Refactoring Day 20 : Extract Subclass

_This refactoring is useful when you have methods on a base class that are not shared amongst all classes and needs to be pushed down into it’s own class._

```C#
public class Registration
{
    public NonRegistrationAction Action { get; set; }
    public decimal RegistrationTotal { get; set; }
    public string Notes { get; set; }
    public string Description { get; set; }
    public DateTime RegistrationDate { get; set; }
}
```

There is something that we’ve realized after working with this class. We are using it in two different contexts.

The properties NonRegistrationAction and Notes are only ever used when dealing with a NonRegistration which is used to track a portion of the system that is slightly different than a normal registration.

Noticing this, we can extract a subclass and move those properties down into the NonRegistration class where they more appropriately fit.

```C#
public class Registration
{
    public decimal RegistrationTotal { get; set; }
    public string Description { get; set; }
    public DateTime RegistrationDate { get; set; }
}

public class NonRegistration : Registration
{
    public NonRegistrationAction Action { get; set; }
    public string Notes { get; set; }
}
```

## Refactoring Day 21 : Collapse Hierarchy

A Collapse Hierarchy refactoring would be applied when you realize you no longer need a subclass. When this happens it doesn’t really make sense to keep your subclass around if it’s properties can be merged into the base class and used strictly from there.

```C#
public class Website
{
    public string Title { get; set; }
    public string Description { get; set; }
    public IEnumerable<Webpage> Pages { get; set; }
}

public class StudentWebsite : Website
{
    public bool IsActive { get; set; }
}
```

Here we have a subclass that isn’t doing too much. It just has one property to denote if the site is active or not. At this point maybe we realize that determing if a site is active is something we can use across the board so we can collapse the hierarchy back into only a Website and eliminate the StudentWebsite type.

```C#
public class Website
{
    public string Title { get; set; }
    public string Description { get; set; }
    public IEnumerable<Webpage> Pages { get; set; }
    public bool IsActive { get; set; }
}
```

## Refactoring Day 22 : Break Method

> This refactoring is kind of a meta-refactoring in the fact that it’s just extract method applied over and over until you decompose one large method into several smaller methods.

Below we have the AcceptPayment method that can be decomposed multiple times into distinct methods.

```C#
public class CashRegister
{
    public CashRegister()
    {
        Tax = 0.06m;
    }
   
    private decimal Tax { get; set; }
   
    public void AcceptPayment(Customer customer, IEnumerable<Product> products,
                                           decimal payment)
    {
        decimal subTotal = 0m;
        foreach (Product product in products)
        {
            subTotal += product.Price;
        }

        foreach(Product product in products)
        {
            subTotal -= product.AvailableDiscounts;
        }
  
        decimal grandTotal = subTotal * Tax;
  
        customer.DeductFromAccountBalance(grandTotal);
    }
}

public class Customer
{
    public void DeductFromAccountBalance(decimal amount)
    {
        // deduct from balance
    }
}

public class Product
{
    public decimal Price { get; set; }
    public decimal AvailableDiscounts { get; set; }
}
```

As you can see the AcceptPayment method has a couple of things that can be decomposed into targeted methods. So we perform the Extract Method refactoring a number of times until we come up with the result:

```C#
public class CashRegister
{
    public CashRegister()
    {
        Tax = 0.06m;
    }

    private decimal Tax { get; set; }
    private IEnumerable<Product> Products { get; set; }

    public void AcceptPayment(Customer customer, IEnumerable<Product> products,
                                        decimal payment)
    {
        decimal subTotal = CalculateSubtotal();

        subTotal = SubtractDiscounts(subTotal);

        decimal grandTotal = AddTax(subTotal);

        SubtractFromCustomerBalance(customer, grandTotal);
    }

    private void SubtractFromCustomerBalance(Customer customer, decimal grandTotal)
    {
        customer.DeductFromAccountBalance(grandTotal);
    }

    private decimal AddTax(decimal subTotal)
    {
        return subTotal * Tax;
    }

    private decimal SubtractDiscounts(decimal subTotal)
    {
        foreach(Product product in Products)
        {
            subTotal -= product.AvailableDiscounts;
        }
        return subTotal;
    }

    private decimal CalculateSubtotal()
    {
        decimal subTotal = 0m;
        foreach (Product product in Products)
        {
            subTotal += product.Price;
        }
        return subTotal;
    }
}

public class Customer
{
    public void DeductFromAccountBalance(decimal amount)
    {
        // deduct from balance
    }
}

public class Product
{
    public decimal Price { get; set; }
    public decimal AvailableDiscounts { get; set; }
}   
```

## Refactoring Day 23 : Introduce Parameter Object

Sometimes when working with a method that needs several parameters it becomes difficult to read the
method signature because of five or more parameters being passed to the method like so:

```C#
public class Registration
{
    public void Create(decimal amount, Student student,
                         IEnumerable<Course> courses, decimal credits)
    {
        // do work
    }
}
```

It’s useful to create a class who’s only responsibility is to carry parameters into the method. This helps make the code more flexible because to add more parameters, you need only to add another field to the parameter object.

```C#
public class RegistrationContext
{
    public decimal Amount { get; set; }
    public Student Student { get; set; }
    public IEnumerable<Course> Courses { get; set; }
    public decimal Credits { get; set; }
}

public class Registration
{
    public void Create(RegistrationContext registrationContext)
    {
        // do work
    }
}
```

## Refactoring Day 24 : Remove Arrowhead Antipattern

> The arrowhead antipattern is when you have nested conditionals so deep that they form an arrowhead of code.

```C#
public class Security
{
    public ISecurityChecker SecurityChecker { get; set; }
   
    public Security(ISecurityChecker securityChecker)
    {
        SecurityChecker = securityChecker;
    }
    
    public bool HasAccess(User user, Permission permission,
                                IEnumerable<Permission> exemptions)
    {
        bool hasPermission = false;
        if (user != null)
        {
            if (permission != null)
            {
                if (exemptions.Count() == 0)
                {
                    if (SecurityChecker.CheckPermission(user, permission) ||
                                         exemptions.Contains(permission))
                    {
                        hasPermission = true;
                    } 
                }
            }
        }
    
        return hasPermission;
    }
}
```

Refactoring away from the arrowhead antipattern is as simple as swapping the conditionals to leave the method as soon as possible. Refactoring in this manner often starts to look like Design By Contract checks to evaluate conditions before performing the work of the method. Here is what this same method might look like after refactoring.

```C#
public class Security
{
    public ISecurityChecker SecurityChecker { get; set; }
   
    public Security(ISecurityChecker securityChecker)
    {
        SecurityChecker = securityChecker;
    }
   
    public bool HasAccess(User user, Permission permission,
                                IEnumerable<Permission> exemptions)
    {
        if (user == null || permission == null)
            return false;
        if (exemptions.Contains(permission))
            return true;
        return SecurityChecker.CheckPermission(user, permission);
    }
}
```

As you can see, this method is much more readable and maintainable going forward. It’s not as difficult to see all the different paths you can take through this method.

## Refactoring Day 25 : Introduce Design By Contract checks

**Design By Contract or DBC defines that methods should have defined input and output verifications.**

In our example here, we are working with input parameters that may possibly be null. As a result a NullReferenceException would be thrown from this method because we never verify that we have an instance. During the end of the method, we don’t ensure that we are returning a valid decimal to the consumer of this method and may introduce methods elsewhere.

```C#
public class CashRegister
{
    public decimal TotalOrder(IEnumerable<Product> products, Customer customer)
    {
        decimal orderTotal = products.Sum(product => product.Price);

        customer.Balance += orderTotal;

        return orderTotal;
    }
}
```

The changes we can make here to introduce DBC checks is pretty easy. First we will assert that we don’t have a null customer, check that we have at least one product to total. Before we return the order total we will ensure that we have a valid amount for the order total. If any of these checks fail in this example we should throw targeted exceptions that detail exactly what happened and fail gracefully rather than throw an obscure NullReferenceException.

```C#
public class CashRegister
{
    public decimal TotalOrder(IEnumerable<Product> products, Customer customer)
    {
        if (customer == null)
            throw new ArgumentNullException("customer", "Customer cannot be null");
        if (product.Count() == 0)
            throw new ArgumentException("Must have at least one product to total",
                                     "products");
        decimal orderTotal = products.Sum(product => product.Price);
        
        customer.Balance += orderTotal;
        
        if (orderTotal == 0)
            throw new ArgumentOutOfRangeException("orderTotal",
                                            "Order Total should not be zero");
        return orderTotal;
}
```

It does add more code to the method for validation checks and you can go overboard with DBC, but I think in most scenarios it is a worthwhile endeavor to catch sticky situations. It really stinks to chase after a NullReferenceException without detailed information.

## Refactoring Day 26 : Remove Double Negative

> This type of code does the most damage because of the assumptions made on it. Assumptions lead to incorrect maintenance code written, which in turn leads to bugs

```C#
public class Order
{
    public void Checkout(IEnumerable<Product> products, Customer customer)
    {
        if (!customer.IsNotFlagged)
        {
            // the customer account is flagged
            // log some errors and return
            return;
        }

        // normal order processing
    }    
}

public class Customer
{
    public decimal Balance { get; private set; }

    public bool IsNotFlagged
    {
        get { return Balance < 30m; }
    }
    
}
```

As you can see the double negative here is difficult to read because we have to figure out what is positive state of the two negatives. The fix is very easy. If we don’t have a positive test, add one that does the double negative assertion for you rather than make sure you get it correct.

```C#
public class Order
{
    public void Checkout(IEnumerable<Product> products, Customer customer)
    {
        if (customer.IsFlagged)
        {
            // the customer account is flagged
            // log some errors and return
            return;
        }

        // normal order processing
    }
}

public class Customer
{
    public decimal Balance { get; private set; }

    public bool IsFlagged
    {
        get { return Balance >= 30m; }
    }
}
```

## Refactoring Day 27 : Remove God Classes

* Often these classes will be suffixed with either `“Utils”` or `“Manager”`
* Classes with multiple grouped pieces of functionality
* Another good indicator of a God class is methods grouped together with using statements or comments into seperate roles that this one class is performing.

```C#
public class CustomerService
{
    public decimal CalculateOrderDiscount(IEnumerable<Product> products,
                                            Customer customer)
    {
        // do work
    }
   
    public bool CustomerIsValid(Customer customer, Order order)
    {
        // do work
    }

    public IEnumerable<string> GatherOrderErrors(IEnumerable<Product> products,
                                            Customer customer)
    {
        // do work
    }

    public void Register(Customer customer)
    {
        // do work
    }

    public void ForgotPassword(Customer customer)
    {
        // do work
    }
}
```

**The refactoring for this is very straight forward. Simply take the related methods and place them in specific classes that match their responsibility. This makes them much finer grained and defined in what they do and make future maintenance much easier.**

```C#
public class CustomerOrderService
{
    public decimal CalculateOrderDiscount(IEnumerable<Product> products,
                                            Customer customer)
    {
        // do work
    }
   
    public bool CustomerIsValid(Customer customer, Order order)
    {
        // do work
    }

    public IEnumerable<string> GatherOrderErrors(IEnumerable<Product> products,
                                            Customer customer)
    {
        // do work
    }
}

public class CustomerRegistrationService
{
    public void Register(Customer customer)
    {
        // do work
    }

    public void ForgotPassword(Customer customer)
    {
        // do work
    }
}
```

## Refactoring Day 28 : Rename boolean method

> Methods with a large number of boolean parameters can quickly get out of hand and can produce unexpected behavior. Depending on the number of parameters will determine how many methods need to be broken out.

```C#
public class BankAccount
{
    public void CreateAccount(Customer customer, bool withChecking,
                    bool withSavings, bool withStocks)
    {
        // do work
    }
}
```

We can make this work a little better simple by exposing the boolean parameters via well named methods and in turn make the original method private to prevent anyone from calling it going forward. Obviously you could have a large number of permutations here and perhaps it makes more sense to refactor to a Parameter object instead.

```C#
public class BankAccount
{
    public void CreateAccountWithChecking(Customer customer)
    {
        CreateAccount(customer, true, false);
    }
   
    public void CreateAccountWithCheckingAndSavings(Customer customer)
    {
        CreateAccount(customer, true, true);
    }
    
    private void CreateAccount(Customer customer, bool withChecking,
                            bool withSavings)
    {
        // do work
    }
}
```


