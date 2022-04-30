# Blazor
The following chapter show some tips and tricks in regards to **Blazor**.

## `CascadingValue` fixed
CascadingValues are used, as the name implies, the cascade a parameter down the hierarchy. The problem with that is, that every child component will listen to changes to the original value. This can get expensive. If your component does not rely on updates, you can pin / fix this value. For that you can use the IsFixed parameter.

❌ **Bad** in case `SomeOtherComponents` does not need update of the cascading value
```razor
<CascadingValue Value="this">
    <SomeOtherComponents>
</CascadingValue>
```

✅ **Good** If the component does **not** need further updates (or the value never changes anyway) we can **fix** the value.
```razor
<CascadingValue Value="this" IsFixed="true">
    <SomeOtherComponents>
</CascadingValue>
```

## `@key` directive
Blazor diff engine determines which elements have to be re-rendered on every render cycle. It does that via sequence numbers. That works in most of cases very well, but adding or removing items in the middle or at the beginning of the list, will lead to re-rendering the whole list instead of the single entry. 

❌ **Bad** in case we entries are added in the middle or at the beginning.
```razor
<ul>
    @foreach (var item in items)
    {
        <li>@item</li>
    }
</ul>
```

✅ **Good** Helping Blazor how it should find the difference between two render cycles.
```razor
<ul>
    @foreach (var item in items)
    {
        @* With the @key attribute we define what makes the element unique
           Based on this Blazor can determine whether or not it has to re-render
           the element
        *@
        <li @key="item">@item</li>
    }
</ul>
```

> 💡 Info: If you want to know more about `@key` [here](https://steven-giesel.com/blogPost/a8772410-847d-4fe7-ba93-3e03ab7748c0) is article about that topic.

## `StateHasChanged` from async call without `InvokeAsync`
In the current state (.net6 / .net7.0 preview 4) Blazor WASM only runs on one thread at the time. Therefore switching context via `await` will still be run on the main UI thread. Once Blazor WASM introduces real threads, code which invokes UI changes from a background thread will throw an exception. For Blazor Server this holds true today, even though the `SynchronizationContext` tries to be as much as possible single threaded.

❌ **Bad** Can lead to exceptions on Blazor Server now or Blazor WASM in the future.
```csharp
protected override void OnInitialized()
{
    base.OnInitialized();
    Timer = new System.Threading.Timer(_ =>
    {
        StateHasChanged();
    }, null, 500, 500);
}
```

✅ **Good** Call `InvokeAsync` to force rendering on the main UI thread.
```csharp
protected override void OnInitialized()
{
    base.OnInitialized();
    Timer = new System.Threading.Timer(_ =>
    {
        InvokeAsync(StateHasChanged);
    }, null, 500, 500);
}
```