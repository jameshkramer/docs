---
title: Removing elements, attributes, and nodes from an XML tree - LINQ to XML
description: Learn how to modify an XML tree, removing elements, attributes, and other types of nodes.
ms.date: 07/20/2015
dev_langs:
  - "csharp"
  - "vb"
ms.assetid: 07dd06d6-1117-4077-bf98-9120cf51176e
---

# Removing elements, attributes, and nodes from an XML tree (LINQ to XML)

You can modify an XML tree, removing elements, attributes, and other types of nodes.

Removing a single element or a single attribute from an XML document is straightforward. However, when removing collections of elements or attributes, you should first materialize a collection into a list, and then delete the elements or attributes from the list. The best approach is to use the <xref:System.Xml.Linq.Extensions.Remove%2A> extension method, which will do this for you.

The main reason for doing this is that most of the collections you retrieve from an XML tree are yielded using deferred execution. If you do not first materialize them into a list, or if you do not use the extension methods, it is possible to encounter a certain class of bugs. For more information, see [Mixed declarative code-imperative code bugs](mixed-declarative-code-imperative-code-bugs.md).

The following methods remove nodes and attributes from an XML tree.

|Method|Description|
|------------|-----------------|
|<xref:System.Xml.Linq.XAttribute.Remove%2A?displayProperty=nameWithType>|Removes an <xref:System.Xml.Linq.XAttribute> from its parent.|
|<xref:System.Xml.Linq.XContainer.RemoveNodes%2A?displayProperty=nameWithType>|Removes the child nodes from an <xref:System.Xml.Linq.XContainer>.|
|<xref:System.Xml.Linq.XElement.RemoveAll%2A?displayProperty=nameWithType>|Removes content and attributes from an <xref:System.Xml.Linq.XElement>.|
|<xref:System.Xml.Linq.XElement.RemoveAttributes%2A?displayProperty=nameWithType>|Removes the attributes of an <xref:System.Xml.Linq.XElement>.|
|<xref:System.Xml.Linq.XElement.SetAttributeValue%2A?displayProperty=nameWithType>|If you pass `null` for value, then removes the attribute.|
|<xref:System.Xml.Linq.XElement.SetElementValue%2A?displayProperty=nameWithType>|If you pass `null` for value, then removes the child element.|
|<xref:System.Xml.Linq.XNode.Remove%2A?displayProperty=nameWithType>|Removes an <xref:System.Xml.Linq.XNode> from its parent.|
|<xref:System.Xml.Linq.Extensions.Remove%2A?displayProperty=nameWithType>|Removes every attribute or element in the source collection from its parent element.|

## Example: Remove a single element, and remove a collection of elements in two ways

This example demonstrates three approaches to removing elements. First, it removes a single element. Second, it retrieves a collection of elements, materializes them using the <xref:System.Linq.Enumerable.ToList%2A?displayProperty=nameWithType> operator, and removes the collection. Finally, it retrieves a collection of elements and removes them using the <xref:System.Xml.Linq.Extensions.Remove%2A> extension method.

For more information on the <xref:System.Linq.Enumerable.ToList%2A> operator, see [Converting data types (C#)](../../csharp/programming-guide/concepts/linq/converting-data-types.md).

```csharp
XElement root = XElement.Parse(@"<Root>
    <Child1>
        <GrandChild1/>
        <GrandChild2/>
        <GrandChild3/>
    </Child1>
    <Child2>
        <GrandChild4/>
        <GrandChild5/>
        <GrandChild6/>
    </Child2>
    <Child3>
        <GrandChild7/>
        <GrandChild8/>
        <GrandChild9/>
    </Child3>
</Root>");
root.Element("Child1").Element("GrandChild1").Remove();
root.Element("Child2").Elements().ToList().Remove();
root.Element("Child3").Elements().Remove();
Console.WriteLine(root);
```

```vb
Dim root As XElement = _
    <Root>
        <Child1>
            <GrandChild1/>
            <GrandChild2/>
            <GrandChild3/>
        </Child1>
        <Child2>
            <GrandChild4/>
            <GrandChild5/>
            <GrandChild6/>
        </Child2>
        <Child3>
            <GrandChild7/>
            <GrandChild8/>
            <GrandChild9/>
        </Child3>
    </Root>
root.<Child1>.<GrandChild1>.Remove()
root.<Child2>.Elements().ToList().Remove()
root.<Child3>.Elements().Remove()
Console.WriteLine(root)
```

This example produces the following output:

```xml
<Root>
  <Child1>
    <GrandChild2 />
    <GrandChild3 />
  </Child1>
  <Child2 />
  <Child3 />
</Root>
```

The first grandchild element has been removed from `Child1`, and all grandchildren elements have been removed from `Child2` and from `Child3`.
