#powershell #get-content #select-string #filter #logging

`Get-Content` is een commando die gebruikt kan worden om tekst uit een bestand te tonen in PowerShell. 

Middels het volgende commando kan bepaalde tekst gefilterd worden, zodat je alleen regels krijgt met een specifieke tekst.

```bash
Get-Content "employees.txt" | Select-String -pattern "Employee 1"
```

Nu wordt alle regels getoond waar de tekst "Employee 1" in voorkomt.

Het is ook mogelijk om meerdere criteria toe te voegen.

```bash
Get-Content "employees.txt" | Select-String -pattern ("Employee 1", "Peter")
```

Het is ook mogelijk om dit te combineren met een log-bestand. Met het volgende commando kan worden gebruikt om een log bestand bij te houden.

```bash
Get-Content "app.log" -Wait -Tail 100 | Select-String -pattern "filter1"
```

Het commando kijkt in het bestand app.log en blijft kijken of er nieuwe tekst in het bestand komt.

# References

- https://collectingwisdom.com/powershell-get-content-filter/
- https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-content
- https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-string

