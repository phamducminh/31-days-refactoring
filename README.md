# 31-days-refactoring
This will keep reminding me refactoring my silly code

## Contents

- [Refactoring Day 1 : Encapsulate Collection](README.md#refactoring-day-1--encapsulate-collection)
- [Refactoring Day 2 : Move Method](README.md#refactoring-day-2--move-method)
- [Refactoring Day 3 : Pull Up Method](README.md#refactoring-day-3--pull-up-method)
- [Refactoring Day 4 : Push Down Method](README.md#refactoring-day-4--push-down-method)

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

