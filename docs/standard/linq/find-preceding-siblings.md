---
title: How to find preceding siblings - LINQ to XML
description: Learn how to use XPathSelectElements along the preceding-sibling axis to find elements that precede a given element, and how to use XNode.ElementsBeforeSelf to find the same elements.
ms.date: 07/20/2015
dev_langs:
  - "csharp"
  - "vb"
ms.assetid: b281ff99-d08a-43d0-bea1-eff831b2f8ae
---

# How to find preceding siblings (LINQ to XML)

This article shows how to use XPathSelectElements along the preceding-sibling axis to find elements that precede a given element, and how to use XNode.ElementsBeforeSelf to find the same elements.

Note that the results of both <xref:System.Xml.XPath.Extensions.XPathSelectElements%2A> and <xref:System.Xml.Linq.XNode.ElementsBeforeSelf%2A?displayProperty=nameWithType> are in document order.

## Example: Use the `preceding-sibling` axis to find sibling elements that precede an element

The following example finds the `FullAddress` element in XML document [Sample XML file: Customers and orders](sample-xml-file-customers-orders), and then retrieves the previous elements in two different ways. It then compares the results and finds them identical. The XPath expression used in XPathSelectElements is `preceding-sibling::*`.

```csharp
XElement co = XElement.Load("CustomersOrders.xml");

XElement add = co.Element("Customers").Element("Customer").Element("FullAddress");

// LINQ to XML query
IEnumerable<XElement> list1 = add.ElementsBeforeSelf();

// XPath expression
IEnumerable<XElement> list2 = add.XPathSelectElements("preceding-sibling::*");

if (list1.Count() == list2.Count() &&
        list1.Intersect(list2).Count() == list1.Count())
    Console.WriteLine("Results are identical");
else
    Console.WriteLine("Results differ");
foreach (XElement el in list2)
    Console.WriteLine(el);
```

```vb
Dim co As XElement = XElement.Load("CustomersOrders.xml")
Dim add As XElement = co.<Customers>.<Customer>. _
        <FullAddress>.FirstOrDefault

' LINQ to XML query
Dim list1 As IEnumerable(Of XElement) = add.ElementsBeforeSelf()

' XPath expression
Dim list2 As IEnumerable(Of XElement) = add.XPathSelectElements("preceding-sibling::*")

If list1.Count() = list2.Count() And _
        list1.Intersect(list2).Count() = list1.Count() Then
    Console.WriteLine("Results are identical")
Else
    Console.WriteLine("Results differ")
End If
For Each el As XElement In list2
    Console.WriteLine(el)
Next
```

This example produces the following output:

```output
Results are identical
<CompanyName>Great Lakes Food Market</CompanyName>
<ContactName>Howard Snyder</ContactName>
<ContactTitle>Marketing Manager</ContactTitle>
<Phone>(503) 555-7555</Phone>
```

## See also

- [LINQ to XML for XPath Users (Visual Basic)](../../visual-basic/programming-guide/concepts/linq/linq-to-xml-for-xpath-users.md)
