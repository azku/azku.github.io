---
layout: post
title:  "Get object reference based on a product instance when walking a tree on CATIA"
author: azku
tags: [catia, automation]
categories: []
---

When iterating through the tree in CATIA, it is very common the need to operate with geometrical elements at instance level. 

A very common task is copying bodies from an instance to another keeping the link. 
Considering that various instances might point to the same part, getting the reference to the specific instance it's key in order to maintain positions.

At any instance of the loop, having the product instance and the element we want to copy, here is how we might create the reference:

{% highlight vb %}

Function CreateReferenceFromProductInstance(oProduct As ProductStructureTypeLib.Product, 
                                            objectToReference As Object)
            Dim oHelperProduct
            Dim strRef As Text.StringBuilder = New Text.StringBuilder()
            Dim root As ProductStructureTypeLib.Product = oProduct.Application.ActiveDocument.Product
            Dim osel = oProduct.Application.ActiveDocument.Selection

            'The left part of the reference
            osel.Clear()
            osel.Add(objectToReference)
            oHelperProduct = osel.FindObject("CATIAPart")
            strRef.Append(objectToReference.Name)
            If TypeOf oHelperProduct Is MECMOD.HybridBody Then
                Do While TypeOf oHelperProduct.Parent Is MECMOD.HybridBodies
                    strRef.Insert(0, "/") : strRef.Insert(0, oHelperProduct.Name)
                    oHelperProduct = oHelperProduct.Parent()
                Loop
                strRef.Insert(0, "/") : strRef.Insert(0, oHelperProduct.Name)
            ElseIf TypeOf objectToReference.Parent Is INFITF.Collection Or TypeOf objectToReference Is MECMOD.HybridShape Then
                strRef.Insert(0, "/") : strRef.Insert(0, objectToReference.Parent.parent.Name)
            Else
                strRef.Insert(0, "/") : strRef.Insert(0, objectToReference.Parent.Name)
            End If
            strRef.Insert(0, "!")

            'The right part of the reference
            osel.Clear()
            osel.Add(oHelperProduct)
            oHelperProduct = osel.FindObject("CATIAProduct")
            Do While oHelperProduct IsNot Nothing
                strRef.Insert(0, "/")
                If oHelperProduct.PartNumber = oProduct.PartNumber And 
                   TypeOf oProduct.Parent Is ProductStructureTypeLib.Products Then
                    strRef.Insert(0, oProduct.Name)
                Else
                    strRef.Insert(0, oHelperProduct.Name)
                End If
                If oHelperProduct.Name = root.Name Then
                    Exit Do
                End If
                osel.Add(oHelperProduct.Parent)
                oHelperProduct = osel.FindObject("CATIAProduct")
            Loop

            Return root.CreateReferenceFromName(strRef.ToString())
    End Function
    
{% endhighlight %}

The returned reference can then be added to a selection object, in order to copy the specific instance.
