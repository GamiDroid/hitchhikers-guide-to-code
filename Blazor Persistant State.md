#blazor #razor #dotnet #state

In een Blazor applicatie wil je soms dat een bepaalde *state* door je gehele of een deel van de applicatie bewaard blijft en dat de applicatie reageert op een verandering van deze *state*. 

Een manier om dit op te lossen zijn *StateProviders*.
# StateProvider

In het onderstaande voorbeeld wordt een *TenantId* state bewaard. Wanneer de waarde van de *TenantId* wijzigd, worden alle luisteraars op de hoogte gebracht.

```csharp
public class TenantStateProvider
{
    private string? _tenantId;

	// Set the current tenant id.
	// Notify the listeners that the tenant has changed.
    public void SetTenantId(string? tenantId)
    {
        if (_tenantId != tenantId)
        {
            _tenantId = tenantId;
            NotifyTenantStateChanged(tenantId);
        }
    }

	// Get the current tenant id
    public string? GetTenantId()
    {
        return _tenantId;
    }

	// The listening actions
    public Action<string?>? TenantStateChanged;

	// Invoke the listening actions.
    private void NotifyTenantStateChanged(string? tenantId)
    {
        TenantStateChanged?.Invoke(tenantId);
    }
}
```

Voeg vervolgens de *StateProvider* toe aan de services collection.

```csharp
var builder = WebApplication.CreateBuilder(args);

... 

builder.Services.AddSingleton<TenantStateProvider>();

...

```

Nu zou je in iedere willekeurige razor component kunnen acteren op een wijziging van de state. 

```csharp
@implements IDisposable
@inject TenantStateProvider _tenantStateProvider

<h1>@_tenantId</h1>

@code {

    private string? _tenantId;

    protected override void OnInitialized()
    {
        _tenantStateProvider.TenantStateChanged += OnTenantStateChanged;
    }

    private void OnTenantStateChanged(string? newTenantId)
    {
        _ = InvokeAsync(() =>
        {
            _tenantId = newTenantId;
            StateHasChanged();
        });
    }

    public void Dispose()
    {
        _tenantStateProvider.TenantStateChanged -= OnTenantStateChanged;
    }
}
```

De bovenstaande implementatie van een razor component zou je natuurlijk op verschillende plekken in de applicatie gebruiken om te acteren op de state verandering.
# CascadingState

Wanneer de applicatie door de gehele applicatie afhankelijk is van de state is het verstandig om een *CascadingState* component te maken. Deze geeft de *state* door aan alle onderliggende componenten.

Om dit te realiseren wordt een [CascadingValue](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/cascading-values-and-parameters?view=aspnetcore-8.0#cascadingvalue-component) component gebruikt. 

```csharp
@* filename: CascadingTenantState *@
@implements IDisposable
@inject TenantStateProvider _tenantStateProvider

<CascadingValue Name="TenantId" Value="@_tenantId" ChildContent="@ChildContent" />

@code {

    private string? _tenantId;

    [Parameter]
    public RenderFragment? ChildContent { get; set; }

    protected override void OnInitialized()
    {
        _tenantStateProvider.TenantStateChanged += OnTenantStateChanged;
    }

    private void OnTenantStateChanged(string? newTenantId)
    {
        _ = InvokeAsync(() =>
        {
            _tenantId = newTenantId;
            StateHasChanged();
        });
    }

    public void Dispose()
    {
        _tenantStateProvider.TenantStateChanged -= OnTenantStateChanged;
    }
}

```

Deze wordt vervolgens om de *RouteView* geplaatst. In *.NET8* vindt je dit in de *Routes.razor* bestand. In eerdere versies van blazor *App.razor*.

```csharp
<Router AppAssembly="typeof(Program).Assembly">
    <Found Context="routeData">
        <CascadingTenantState>
			<RouteView RouteData="routeData" 
					DefaultLayout="typeof(Layout.MainLayout)" />
            <FocusOnNavigate RouteData="routeData" Selector="h1" />
        </CascadingTenantState>
    </Found>
</Router>
```

```csharp
<h1>@TenantId</h1>

@code {
	[CascadingParameter(Name = "TenantId")] 
	public string? TenantId { get; set; } 
}
```

# Links
- [Microsoft Learn - Blazor - State management](https://learn.microsoft.com/en-us/aspnet/core/blazor/state-management)
- [Microsoft Learn - ASPNET - Session and State management](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/app-state)



