---
layout: post
title: The Family Tree of XSLT SharePoint WebParts
date: 2013-03-11 09:26:00.000000000 -07:00
type: post
published: true
status: publish
categories:
- C#
- SharePoint
- SharePoint 2010
- SharePoint 2013
tags: []
---
I had some trouble with some SharePoint webparts and what they seemed to have in common was a base type called the BaseXsltDataWebPart.

I could resolve it on a case by case basis but ideally I really wanted to know all the types that derived at some point from the BaseXsltDataWebPart. So I wrote this:     


``` csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
namespace DerivedTypes {
    class Program {
        static void Main(string[] args) {
            Type sample1 = typeof(Microsoft.SharePoint.Publishing
                  .WebControls.SummaryLinkWebPart);

            Type baseType = typeof(Microsoft.SharePoint
                        .WebPartPages.BaseXsltDataWebPart);

            foreach (var s in GetDerivedTypes(sample1,baseType)
                  .Union(GetDerivedTypes(baseType,baseType))) {
                Console.WriteLine(s.Name);
            }
        }

        public static IEnumerable GetDerivedTypes(Type webpartSampleType, 
          Type baseType) { 
            var inheritted = Assembly.GetAssembly(webpartSampleType)
                        .GetExportedTypes()
                        .Where(type => type.IsSubclassOf(baseType));

            return inheritted;
        }
    }
}
```

It yields a list of all the SharePoint webparts contained in Microsoft.SharePoint and Microsoft.SharePoint.Publishing assemblies that inherit from the base xslt webpart that was causing the issue.

Interestingly enough if I call .GetTypes() on the assembly it blows up with a reflection error, the stack trace points to dotfuscator, so presumably some sort of anti-ildasm feature is enabled for reflecting private types, still I only needed the publicly surfaced web parts anyway so not a major problem.
