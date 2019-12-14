# 31-days-refactoring
This will keep reminding me refactoring my silly code

## Contents

- [Refactoring Day 1 : Encapsulate Collection](README.md#refactoring-day-1-encapsulate-collection)

---

## Refactoring Day 1 : Encapsulate Collection

* Not expose a full collection to consumers of a class
* Only expose the collection as something you can iterate over without modifying the collection
* Using this can ensure that consumers do not mis-use your collection and introduce bugs into the code

```java
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



