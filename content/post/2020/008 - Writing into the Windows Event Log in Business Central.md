---
title: Writing into the Windows Event Log in Business Central
date: 2020-11-01T14:59:00.4859120+01:00
tags: ["AL", "dotNet"]
draft: false
---

Recently, I had to dump some information somewhere to debug an error. Instead of writing it into a file, I gave the Windows Event Log a try this time. 
<!--more-->

Thanks to the .NET Framework, the implementation is quite straightforward. The user Oki posted a reference implementation of it in C/SIDE on [mibuso.com](https://forum.mibuso.com/discussion/65841/how-to-write-to-eventlog-from-nav-using-dotnet).

But C/SIDE is dead and I wanted to know how to implement it in a pure AL environment.

## **Disclaimer**

Only Business Central OnPrem is supporting the DotNet datatype. The code shown here will not work in the cloud version of BC.

To enable DotNet in your application you need to set `target` to `onPrem` in your `app.json`.

```json
{
    ...
    "target": "OnPrem"
    ...
}
```

## **Implementation**

### **The .NET Package**

Before being able to use .NET in your app, you need to declare all needed Assemblies in a .NET package.

**Example**
```al
dotnet
{
    assembly(<NameOfTheAssembly>)
    {
        type(<ClassName>; <YourAliasForIt>) { }
    }
}
```

For writing into the Event Log, we need the class `EventLog` and the enum `EventLogEntryType` of the assembly `System`:

```al
dotnet
{
    assembly(System)
    {
        type(System.Diagnostics.EventLog; WinEventLog) { }
        type(System.Diagnostics.EventLogEntryType; WinEventLogEntryType) { }
    }
}
```

### **Event Log Management Codeunit**

After declaring the .NET package, you can use these defined .NET types via the specified aliases.

```al
codeunit 50100 "Windows-EventLog Management"
{
    procedure WriteEventlog()
    var
        EventLog: DotNet WinEventLog;
        EventLogEntryType: DotNet WinEventLogEntryType;
    begin
        // ...
    end;
}
```

The Windows Event Log has different logs such as Application, System, Security, and so on. Each entry belongs to one of these logs. Furthermore, a log entry has a source. Sources can be any free text, but usually, it's the name of the Application. Besides the Source, custom logs need to be created first—except for the Windows standard ones.
To keep it simple, I decided to write into the `Application` log. The source I set to `MicrosoftDynamicsNav`.

```al
codeunit 50100 "Windows-EventLog Management"
{
    procedure WriteEventlog()
    var
        EventLog: DotNet WinEventLog;
        EventLogEntryType: DotNet WinEventLogEntryType;
        EventLogName: Label 'Application';
        EventLogSource: Label 'MicrosoftDynamicsNav';
    begin
        EventLog := EventLog.EventLog(EventLogName);
        EventLog.Source := EventLogSource;
    end;
}
```

After the EventLog class is instantiated, we can use the function `WriteEntry` to create the Event Log Entry. That function takes a string as the log message and the Entry Type as a parameter. Entry Types can be `Information`, `Warning` or `Error`.

```al
codeunit 50100 "Windows-EventLog Management"
{
    procedure WriteEventlog()
    var
        EventLog: DotNet WinEventLog;
        EventLogEntryType: DotNet WinEventLogEntryType;
        EventLogName: Label 'Application';
        EventLogSource: Label 'MicrosoftDynamicsNav';
    begin
        EventLog := EventLog.EventLog(EventLogName);
        EventLog.Source := EventLogSource;

        EventLog.WriteEntry('Hello World', EventLogEntryType.Information);
        EventLog.Dispose;
    end;
}
```

### **Checking the Event Log**

After using the function we just created, you will find the following entry in your Event Log:

```powershell
PS C:\> Get-EventLog -LogName Application -Newest 1 -Before '2020-11-01 14:47' | Format-Table -wrap

   Index Time          EntryType   Source                 InstanceID Message
   ----- ----          ---------   ------                 ---------- -------
    1594 Nov 01 14:32  Information MicrosoftDynamicsNav            0 Hello World
```


## **Code on Github**

You will find my implementation on my GitHub profile.

https://github.com/codeunitone/BC-EventLog-Management

## **Further Information**

* [Implementation on GitHub](https://github.com/codeunitone/BC-EventLog-Management)
* [Getting started with Microsoft .NET Interoperability from AL](https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/devenv-get-started-call-dotnet-from-al)
* [EventLog](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.eventlog?view=netframework-4.8)
* [EventLogEntryType](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.eventlogentrytype?view=netframework-4.8)   