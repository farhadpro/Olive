# Entity: Database lifecycle events
The `Entity` class provides **object lifecycle** methods and events that you can hook into for implementing custom business logic in your applications.

## Saving an entity in the database
When you save an entity in the database, whether for insert or update operation, the following events will be invoked automatically by the framework. They run in the following order:

#### OnValidating()
This is invoked automatically as the first step when you try to save an entity in the database using the `Database.Save()` command. This executes before calling the `Validate()` method. You should not use it to implement your validation logic. Instead, use this to do any last-minute object modifications, such as initializing complex values.

For example:
```csharp
class Customer : GuidEntity
{
    public string OfficialName {get; set;}
    public string FriendlyName {get; set;}

    protected override async Task OnValidating()
    {
        await base.OnValidating();
        
        if (FriendlyName.IsEmpty() && OfficialName.HasValue())
           FriendlyName = OfficialName.TrimEnd(" Ltd", caseSensitive: false);
    }
}
```
        
#### Validate()
This method is invoked by the framework when saving an entity, right after `OnValidating()` call is completed.
You should override this method in your business entity types in order to provide custom validation logic.
It expects a `ValidationException` to be thrown in case of an invalid state. Should that be the case, the saving operation will be terminated.

For example:
```csharp
class Customer : GuidEntity
{
    ...

    protected override async Task Validate()
    {
        await base.Validate();
        
        if (FriendlyName.IsEmpty() && OfficialName.HasValue())
           throw new ValidationException("When official name is specified, friendly name must also be specified.");
    }
}
```

#### OnSaving()
This event is raised just before an entity is saved in the data repository. It is invoked right after a successful call to `Validate()`.
You can override this method to implement custom business logic in some rare scenarios. 

> Note: You should not normally use this to amend the state of the object, because in that case the validation logic will not be able to catch an error in your modifications. In such cases, use `OnValidating()` instead.

This method contains an argument of type `CancelEventArgs` which allows you to cancel the save operation. For example:
```csharp
class Customer : GuidEntity
{
    ...

    protected override async Task OnSaving(CancelEventArgs e)
    {
        await base.OnSaving(e);
        
        if (/*some scenario*/)
           e.Cancel = true;
    }
}
```

#### OnSaved()
This event is raised after this instance is saved in the database. You can use it for simple workflows. 
It takes in an argument of type `SaveEventArgs` with a property named `Mode` which is an enum with the options of `Insert` and `Update`.

For example:
```csharp
class Customer : GuidEntity
{
    ...

    protected override async Task OnSaved(SaveEventArgs e)
    {
        await base.OnSaved(e);
        
        if (e.Mode == SaveMode.Insert)
        {
            // A new customer is just inserted. Notify the customer service team:
            Notifications.ReportRegistration(this);
        }
    }
}
```


, `OnDeleting()`, `OnDeleted()`. To implement custom business logic related to the lifecycle events of a