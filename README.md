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
- [Refactoring Day 9 : Extract Interface](README.md#refactoring-day-9--extract-interface)
- [Refactoring Day 10 : Extract Method](README.md#rsefactoring-day-10--extract-method)
- [Refactoring Day 11 : Switch to Strategy](README.md#refactoring-day-11--switch-to-strategy)

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






