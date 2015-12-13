--- 
layout: post
title: "Emulating document chaining in Qlik Sense"
time: '23:12'
---


In QlikView document chaining is drilling down from one application to another automatically transferring selections.

The current version of Qlik Sense (2.0.1 beta) do not support document chaining yet, but with some efforts, we can achieve a very close approximation of that functionality.

To make a sample I've created a couple of applications with identical models (with load scripts copied from QlikView Script editor's `Test script`).
First application, AppChainingMain, would play the role of a master dashboard (nothing fancy, just one sheet with a Pie Chart and couple of list boxes to filter data)

<img src="/images/app-chaining-main.png" alt="AppChainingMain" width="700">

Second application, AppChainingDetail, is similarly basic in design and consist of one table with transactions details

<img src="/images/app-chaining-detail.png" alt="AppChainingDetail" width="700">


Our goal will be to add the drill-down feature from AppChainingMain into the AppChainingDetail with current selecitons transferred.


The first problem to solve is to find a method to open Qlik Sense application with some predefined selections. Qlik Sense Single Integration API is ideally suited for that purpose. 
For example, such URL could be used to open sheet with id `rMmqj` of application `AppChainingDetail` with selected value `B `in field `Dim1` and selected values `Y` and `Z` in field `Dim3`


    http://localhost:4848/single?appid=AppChainingDetail&amp;sheet=rMmqj&amp;opt=currsel&amp;select=clearAll&amp;select=Dim1,B&amp;select=Dim3,Y,Z

The first part of that URL string is somewhat static or better to say application specific. That part of URI I separate into a variable
vG.ChainUriPrefix with definition

    http://localhost:4848/single?appid=AppChainingDetail&sheet=rMmqj&opt=currsel&select=clearAll


Truly dynamic parts of URIare these

    &select=Dim1,B
    &select=Dim3,Y,Z

These URI query fragments code field selections to make in AppChainingDetail and should be made automatically on a base of current selections in AppChainingMain.

To create these fragments more easily I add variable `vG.SelectionQuery` in variable editor with definition

    If(GetSelectedCount($1)=0,Null(),'&select=' & '$1,' & Concat(distinct $1,','))

With these two variables in place, we can create Text & Image object and use these variables in expression to check a resulting URL Expression in question would have such definition:
 
    vG.ChainUrlPrefix & $(vG.SelectionQuery(Dim1)) & $(vG.SelectionQuery(Dim2)) & $(vG.SelectionQuery(Dim3))

That mean we pass selections in fields Dim1, Dim2 and Dim3

Making various selections we could see how that URL is changing. Unfortunately, I do not know the easy way to copy text from Text & Image object. (The not so easy way is to find it by `localhost:4848` substring in Code Inspector). So now we can copy that text, paste it into the browsers address string and see if all works as intended. In my case, it does and AppChainingDetail opens with same selections as in AppChainingMain.

Next step is to actually add some control with that drill-down feature to AppChaingingMain application. Text & Image object has a link property, but it cannot be linked to an expression, so, for now, it is not suited for that puprose.

So basically, we should find some extension wich is capable of opening dynamically formed URL. I've tried  [Sheet Navigation + Actions for Qlik Sense](http://branch.qlik.com/projects/showthread.php?647-Sheet-Navigation-Actions-for-Qlik-Sense) and it works well for me. So we create extension button with label "Open details", navigation action "Open website" and formula in Website URL same as in Text & image object before, but prepended by equal sign. Clik on newly added button and sheet from second application will be opened:

<img src="/images/app-chaining-drilldown.png" alt="Drill down" width="700">


Take a notice that though detail sheet has a filter panel, application panel is unavailable. So for example navigation to other sheets in detail app should be implemented by some other means - by same "Sheet Navigation + Actions" or some other extension (maybe drop-down menu extension could be utilized to implement navigation between sheets of both applications)



Sample applications ([AppChainingMain](downloads/AppChainingMain.qvf), [AppChainingDetail](downloads/AppChainingDetail.qvf)) are available for download, Sheet Navigation + Actions extension should be already installed for drill-down to work
