
Show free vs available physical memory in GBs:
```
Get-CimInstance Win32_OperatingSystem | Select-Object @{Name="Total Memory (GB)";Expression={$_.TotalVisibleMemorySize / 1024 / 1024}}, @{Name="Free Memory (GB)";Expression={$_.FreePhysicalMemory / 1024 / 1024}}
```

Show top N (20 in this case) processes consuming memory:
```
Get-Process | Sort-Object WorkingSet -Descending | Select-Object ProcessName, @{Name="Memory (GB)";Expression={$_.WorkingSet / 1GB}} -First 20
```

