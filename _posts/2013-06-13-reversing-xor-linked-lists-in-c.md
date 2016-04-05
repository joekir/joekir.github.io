---
layout: post
title: Reversing XOR linked lists in C#
date: 2013-06-13 19:30:00.000000000 -07:00
type: post
published: true
status: publish
categories:
- C#
- datastructures and algorithms
- Pointers
- unsafe keyword
- XOR linked list
tags: []
---
In C# I generally don't explicitly write unsafe code (i.e I don't explicitly use the unsafe keyword). However there are some datastructures and algorithm problems that for an optimal solution require that you access raw pointers.I was looking through some of these problems recently and couldn't find any other implementations of an XOR linked list and reversing it in C#. So I thought I'd post it.

<div><a href="http://en.wikipedia.org/wiki/XOR_linked_list" target="_blank">The XOR linked list</a> is a pretty impressive design in my opinion as it has the storage of a linked list but the functionality of a doubly-linked list (i.e. you can move forwards and backwards). I'm not entirely sure what a valid use case of this datastructure is in modern computing, but there may be some application in preventing eavesdropping, as one cannot determine the value by accessing a pointer directly only when traversing in order.Here's my rough sample code in C#:

{% highlight csharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Runtime.InteropServices;
namespace LinkedLists

public class Program {

  unsafe static void Main(string[] args) {

     int[] values = new int[] { 1, 2, 2, 4, 50, 6, 7, 6, 9, 10, 426,353,2,1 };
     XorListElement* curr = ListCreator.xorLinkedListCreator(values);
     XorListElement* previous = null;
     XorListElement* oldCurr;
     while ((*curr).Value != 0){
       int xorelem = (int)&*(*curr).Xor;
       int previouselem = previous != null ? (int)&previous : 0;
       Console.WriteLine("Element has value: {0}, xorValue: {1}," +
       "previousElem: {2}", curr->Value, xorelem, previouselem);
       oldCurr = curr;
       curr = ListUtils.AdvanceXorPointer(previous, curr->Xor);
       previous = oldCurr;
     }
  }

  unsafe struct ListElement {
    public ListElement* next { get; set; }
    public int Value { get; set; }
  }

  unsafe struct XorListElement {
    public int Value { get; set; }
    public XorListElement* Xor { get; set; }
  }

  unsafe struct ListCreator {
    public static ListElement* linkedListCreator(int[] values) {
      ListElement* curr;
      ListElement* head;
      head = null;
      for (int i = values.Count() - 1; i >= 0; i--){ 
        curr = (ListElement*)Marshal.AllocHGlobal(sizeof(ListElement));
        curr->Value = values[i];
        curr->next = head;
        head = curr;
      }
      curr = head;
      return curr;
    }
  
  public static XorListElement* xorLinkedListCreator(int[] values){

      XorListElement* previous;
      XorListElement* curr;
      XorListElement* next;
      XorListElement* head = null;
      previous = null;
      curr = null;
      bool firstCycle = true;
      for (int i = 0; i < values.Count(); i++) { 
        if (firstCycle) { 
          curr = (XorListElement*)Marshal.AllocHGlobal(sizeof(XorListElement));
          head = curr; 
          firstCycle = false; 
        } 
        curr->Value = values[i];
        next = (XorListElement*)Marshal.AllocHGlobal(sizeof(XorListElement));
        
        //We set the next value to 0 incase at some point its the end of the list.
        //That is the stop condition
        next->Value = 0;
        curr->Xor = ListUtils.AdvanceXorPointer(previous, next);
        previous = curr;
        curr = next;
      }
      return head;
  }
}

Unsafe struct ListUtils {
  public static XorListElement* AdvanceXorPointer(XorListElement* previous, 
                                    XorListElement* prevXORnext) {
    return (XorListElement*)((uint)previous ^ (uint)prevXORnext);
  }

  public static ListElement* head { get; set; }
  public static ListElement* Reverse(ListElement* head) {
    ListElement* temp;
    ListElement* current;
    ListElement* previousNode = null;
    current = head;
    while (current != null) {
      //Update current pointers
      //Save the next pointer
      temp = current->next;
      //Set the next property to be the previous pointer
      current->next = previousNode;
      //Setup the previous and current nodes for the next loop
      previousNode = current;
      current = temp;
    }
    return previousNode;
  }
}
{% endhighlight %}
